# JStorm

## Acker机制

`SpoutCollector.java`
```Java
public List<Integer> sendMsg(String out_stream_id, List<Object> values, Object message_id,
                             Integer out_task_id, ICollectorCallback callback) {
    final long startTime = emitTotalTimer.getTime();
    try {
        boolean needAck = (message_id != null) && (ackerNum > 0);
        Long root_id = getRootId(message_id);
        List<Integer> outTasks;

        if (out_task_id != null) {
            outTasks = sendTargets.get(out_task_id, out_stream_id, values, null, root_id);
        } else {
            outTasks = sendTargets.get(out_stream_id, values, null, root_id);
        }

        List<Long> ackSeq = new ArrayList<>();
        for (Integer t : outTasks) {
            // 根据spout后面的分发节点数，为这个条数据生成不同的ack值
            MessageId msgId;
            if (needAck) {
                // Long as = MessageId.generateId();
                Long as = MessageId.generateId(random);
                msgId = MessageId.makeRootId(root_id, as);
                ackSeq.add(as);
            } else {
                msgId = null;
            }

            TupleImplExt tp = new TupleImplExt(topology_context, values, task_id, out_stream_id, msgId);
            tp.setTargetTaskId(t);
            transfer_fn.transfer(tp);
        }
        sendMsgToAck(out_stream_id, values, message_id, root_id, ackSeq, needAck);
        if (callback != null)
            callback.execute(out_stream_id, outTasks, values);
        return outTasks;
    } finally {
        emitTotalTimer.updateTime(startTime);
    }
}
```

`Acker.java`
```Java
@Override
public void execute(Tuple input) {
    Object id = input.getValue(0);
    AckObject curr = pending.get(id);
    String stream_id = input.getSourceStreamId();
    // 判断STREAMID是否初为始化类型
    if (Acker.ACKER_INIT_STREAM_ID.equals(stream_id)) {
        // 判断此消息ID是否在缓存中
        if (curr == null) {
            curr = new AckObject();

            curr.val = input.getLong(1);
            curr.spout_task = input.getInteger(2);
            // 插入
            pending.put(id, curr);
        } else {
            // bolt's ack first come
            // 更新
            curr.update_ack(input.getValue(1));
            curr.spout_task = input.getInteger(2);
        }

    } else if (Acker.ACKER_ACK_STREAM_ID.equals(stream_id)) {
        // 处理ack
        if (curr != null) {
            curr.update_ack(input.getValue(1));
        } else {
            // two case
            // one is timeout
            // the other is bolt's ack first come
            curr = new AckObject();
            curr.val = input.getLong(1);
            pending.put(id, curr);
        }
    } else if (Acker.ACKER_FAIL_STREAM_ID.equals(stream_id)) {
        // 处理fail
        if (curr == null) {
            // do nothing
            // already timeout, should go fail
            return;
        }
        curr.failed = true;
    } else {
        LOG.info("Unknown source stream, " + stream_id + " from task-" + input.getSourceTask());
        return;
    }
    
    Integer task = curr.spout_task;
    if (task != null) {
        // val的值为异或操作的结果，为0说明spout首先发来的ack值与spout之后的bolt发来的值相同（消息被确认）
        if (curr.val == 0) {
            pending.remove(id);
            List values = JStormUtils.mk_list(id);
            collector.emitDirect(task, Acker.ACKER_ACK_STREAM_ID, values);
        } else {
            if (curr.failed) {
                pending.remove(id);
                List values = JStormUtils.mk_list(id);
                collector.emitDirect(task, Acker.ACKER_FAIL_STREAM_ID, values);
            }
        }
    }

    // add this operation to update acker stats
    collector.ack(input);
    // 判断信息是否超时
    long now = System.currentTimeMillis();
    if (now - lastRotate > rotateTime) {
        lastRotate = now;
        Map<Object, AckObject> tmp = pending.rotate();
        if (tmp.size() > 0) {
            LOG.warn("Acker's timeout item size:{}", tmp.size());
        }
    }

}
```

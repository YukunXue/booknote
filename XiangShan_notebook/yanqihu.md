# Code Review

## 1. Review Flow

* Load  请求流程
* Store 请求流程 
* Probe 请求流程
* Meta  与 Data 同步
* Load  同步
* 并发请求的序列化 （？）

### 1. MissQueue:

Yanqihu:

DCacheWrapper:

1. IMP :

```scala
val missQueue = Module(new MissQueue(edge))
```

2. ARB (ldu <> missQueue) && refill load queue

```scala
for(w <- 0 until LoadPipeWidth) {missReqArb.io.in(w+1) <> ldu(w).io.miss_req}

block_decoupled(missReqArb.io.out, missQueue.io.req, wb.io.block_miss_req)

io.lsu.lsq <> missQueue.io.refill
```

3. tilelink channel-a && channel-b && channel-e

```scala
bus.a <> missQueue.io.mem_acquire
bus.e <> missQueue.io.mem_finish
missQueue.io.probe_req := bus.b.bits.address

block_decoupled(bus.b, probeQueue.io.mem_probe, missQueue.io.probe_block)
```

4. mainPipe <> missQueue ()

```scala
mainPipeReqArb.io.in(MissMainPipeReqPort) <> missQueue.io.pipe_req


missQueue.io.pipe_resp  <> mainPipe.io.miss_resp
```

5. tilelink channel-d

```scala
missQueue.io.mem_grant.valid := false.B
missQueue.io.mem_grant.bits  := DontCare

when(bus.d.bits.opcode === TLMessages.Grant || bus.d.bits.opcode === TLMessages.GrantData){
    missQueue.io.mem_grant  <> bus.d
}
```

6. full signal
```scala
    io.mshrFull := missQueue.io.full
```


Kunminghu:




### 2. MSHR

1. work flow
```

receive req -> send acquire req -> receive grant resp

-> let main pipe do refill and replace -> wait for resp

-> send finish to end the tilelink transaction(before pip_req(?))

```
2.



### 3.MainPipe

1. Meta array
```scala
metaArray.io.write <> mainPipe.io.meta_write

metaReadArb.io.in(MainPipMetaReadPort) <> mainPipe.io.meta_read

mainPipe.io.meta_resp <> metaArray.io.resp(LoadPipelineWidth-1)
``` 

2. Data array

```scala
dataArray.io.write <> mainPipe.io.data_write

dataReadArb.io.in(MainPipeDataReadPort) <> mainPipe.io.data_read

dataArray.io.resp(LoadPipelineWidth-1) <> mainPipe.io.data_resp
```

3. ldu pipe
```scala
ldu(w).io.disable_ld_fast_wakeup := mainPipe.io.disable_ld_fast_wakeup(w)
```

4. missReq
```scala
missReqArb.io.in(MainPipeMissReqPort) <> mainPipe.io.miss_req

//bypass op

mainPipe.io.req.valid := mainPipeReq_valid
mainPipe.io.req.bits  := mainPipeReq_req
```

5. mainPipe_req
```scala
missQueue.io.pipe_resp         <> mainPipe.io.miss_resp
storeReplayUnit.io.pipe_resp   <> mainPipe.io.store_resp
atomicsReplayUnit.io.pipe_resp <> mainPipe.io.amo_resp

probeQueue.io.lrsc_locked_block<> mainPipe.io.lrsc_locked_block
```

6. replace
```scala
mainPipe.io.replace_access(i) <> ldu(i).io.replace_access
```

7. wb
```scala
wb.io.req <> mainPipe.io.wb_req
```

8. status_dup

当前版本多了 tag_write :

s3 写入tag_write

old version:
replace_access -> 传递 miss 的 way && set (input)

new version:
replace_access -> update policy (output)


当前版本的 mainPipe replace机制：

S0:

read meta and tag

S1:

read data // get meta , check hit or miss

(duplicate regs to reduce fanout)

question:

old version : 
        
        1. compare meta not tag ?
        2. core dataArray diff -> dataArray 
        3. mainPipe S3 -> send missQueue

new version : 
        
        1. meta data construct the state of cache about tilelink

              divided meta data into tag && (information)data(ClientMetaData)
        
        2. core dataArray diff -> bankedDataArray 
                    ||1. how to active read -> by var data_read_intend

                    ||2. the function of IO data_read ?

                    ||3. replace algorithm in DCacheWrapper not in MainPipe

        3. mainPipe S2 -> send missQueue ( Follow )


S2: 

select data, return resp if this is a store miss

//select data


S3:
write data/amo 

//do permission checking, write/amo stuff in S3

//change cache internal states(lr/sc counter, tag/data array)


### 3.从refill_queue出发：
```scala
  val bankedDataArray = if(EnableDCacheWPU) Module(new SramedDataArray) else Module(new BankedDataArray)
  val metaArray = Module(new L1CohMetaArray(readPorts = LoadPipelineWidth + 1, writePorts = 2))
  val errorArray = Module(new L1FlagMetaArray(readPorts = LoadPipelineWidth + 1, writePorts = 2))
  val prefetchArray = Module(new L1FlagMetaArray(readPorts = LoadPipelineWidth + 1, writePorts = 2)) // prefetch flag array
  val accessArray = Module(new L1FlagMetaArray(readPorts = LoadPipelineWidth + 1, writePorts = LoadPipelineWidth + 2))
  val tagArray = Module(new DuplicatedTagArray(readPorts = LoadPipelineWidth + 1))

```

几个 core data module

移除refill_pipe需要相应的减少port

1.减少metaArrary的write_port:
```scala
  //----------------------------------------
  // meta array

  // read / write coh meta
  val meta_read_ports = ldu.map(_.io.meta_read) ++
    Seq(mainPipe.io.meta_read)
  val meta_resp_ports = ldu.map(_.io.meta_resp) ++
    Seq(mainPipe.io.meta_resp)
  val meta_write_ports = Seq(
    mainPipe.io.meta_write,
    refillPipe.io.meta_write //remove
  )
  meta_read_ports.zip(metaArray.io.read).foreach { case (p, r) => r <> p }
  meta_resp_ports.zip(metaArray.io.resp).foreach { case (p, r) => p := r }
  meta_write_ports.zip(metaArray.io.write).foreach { case (p, w) => w <> p }

```

## Load Miss Operation:

```
yanqihu:

1. ldu send miss_req  -> miss_Queue ->
               |          |
               |          |
             data  < -- L2 MEM

```

不需要替换块的步骤：

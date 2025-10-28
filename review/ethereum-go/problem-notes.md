## Q：bootnode启动参数是否支持hostname:port
由于区块链部署在容器云上面，所以bootnodes所在的pod地址是变化的，于是每一次启动signer或者member node节点的时候，最好通过bootnode的域名解析到bootnode上面，很可修，直至1.10.13-stable版本，尚未支持bootnode ennode解析dns，只能支持admin.addPeer解析enode的域名

> https://github.com/ethereum/go-ethereum/pull/18524

## Q：由于bootnode不支持域名解析，k8s部署区块链的时候，如何做到双机房部署。
可以通过部署一台虚拟机，上面跑一个普通的节点，然后双边的机房通过该虚拟机完成联通


## Q：区块链执行过程中提示block [xxx] not found
这种情况，作者说往往是由于没有优雅中止geth程序引起的。并且推荐使用以下语句进行优雅关闭

    [user@work ~]$ kill -s SIGQUIT `pidof geth` # Kills it and produces a stacktrace
    [user@work ~]$ kill -s SIGKILL `pidof geth` # Kills it hard
    [user@work ~]$ kill -s SIGINT `pidof geth` # Sends the interrupt signal, triggering graceful shutdown
同时，作者将在PR 21650中，往geth里面增加一个修复block的工具，但是直至1.10.13-stable版本，尚未合并到master分支中。
### reference:
> https://github.com/ethereum/go-ethereum/issues/22763
> https://github.com/ethereum/go-ethereum/pull/21650

## 区块链重组是什么
由于在以太坊网络中，每个节点接收到不同”矿工“节点产生的区块时间不同，因此可能产生首先接收到了一个难度值较低的区块，随后又接收到了一个难度值更高且处于同一高度的新区块，当发生这种这种情况时，便会进行区块链头部的切换，以总难度值最大的区块链为主链。  
注意：由于可能发生区块链重组的情况，因此即便一个新的认证节点被加入，或者被删除，都有可能发生回滚。
### reference
> https://blog.csdn.net/shangsongwww/article/details/90112479

## 同步的时候出现“Discarded delivered header or block, too far away”
在ethereum-go的源码中，[block_fetcher.go](./resource/block_fetcher.go)中的enqueue方法记录了该发生情况。总的来说，这个不是error问题，是因为在获取区块的时候，eth会优先处理与本地链相近的区块和头，所以这个信息在日志中是TRACE级别，这个不是错误。
```go
// enqueue schedules a new header or block import operation, if the component
// to be imported has not yet been seen.
func (f *BlockFetcher) enqueue(peer string, header *types.Header, block *types.Block) {
	var (
		hash   common.Hash
		number uint64
	)
	if header != nil {
		hash, number = header.Hash(), header.Number.Uint64()
	} else {
		hash, number = block.Hash(), block.NumberU64()
	}
	// Ensure the peer isn't DOSing us
	count := f.queues[peer] + 1
	if count > blockLimit {
		log.Debug("Discarded delivered header or block, exceeded allowance", "peer", peer, "number", number, "hash", hash, "limit", blockLimit)
		blockBroadcastDOSMeter.Mark(1)
		f.forgetHash(hash)
		return
	}
	// Discard any past or too distant blocks
	if dist := int64(number) - int64(f.chainHeight()); dist < -maxUncleDist || dist > maxQueueDist {
		log.Debug("Discarded delivered header or block, too far away", "peer", peer, "number", number, "hash", hash, "distance", dist)
		blockBroadcastDropMeter.Mark(1)
		f.forgetHash(hash)
		return
	}
	// Schedule the block for future importing
	if _, ok := f.queued[hash]; !ok {
		op := &blockOrHeaderInject{origin: peer}
		if header != nil {
			op.header = header
		} else {
			op.block = block
		}
		f.queues[peer] = count
		f.queued[hash] = op
		f.queue.Push(op, -int64(number))
		if f.queueChangeHook != nil {
			f.queueChangeHook(hash, true)
		}
		log.Debug("Queued delivered header or block", "peer", peer, "number", number, "hash", hash, "queued", f.queue.Size())
	}
}
```
> https://segmentfault.com/a/1190000017047277

## 同步区块的时候，提示“retrieved hash chain is invalid: invalid new chain”
根据[blockchain.go](./resource/blockchain.go)中的代码判断，这种报错是在同步别的节点的链，并合并本地的链时产生的错误。也就是产生多条链，看情况应该是本地链或者远程链其中有一条产生链空的block，所以我们可以尝试使用debug.setHead("0xblocknumber")的方式重置链的头部.

## Wait, so I can’t run a full node on an HDD?
A: Unfortunately not. Doing a fast sync on an HDD will take more time than you’re willing to wait with the current data schema. Even if you do wait it out, an HDD will not be able to keep up with the read/write requirements of transaction processing on mainnet.

You however should be able to run a light client on an HDD with minimal impact on system resources. If you wish to run a full node however, an SSD is your only option.
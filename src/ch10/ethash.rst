:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch10/ethash.rst

.. _reference-ethash-chapter:

ETHASH 挖矿算法
==============================

把最好的东西留到最后品尝，本书将以太坊的PoW(Proof of work)挖矿算法核心ETHASH算法作为最好的礼物留到最后讲解。掌握了ETHASH算法我们就有能力进行新的、独立的公链开发。这个知识也是最难消化的，需要读者有许多相应的数学知识背景。请让我们一步一步来对照着源代码讲解这部分知识。

ETHASH和比特币PoW的异同
----------------------------

ETHASH这个算法是消耗资源的PoW模式，同比特币共识算法一样，它需要消耗大量的计算资源得出最终的“结论”。但是这个结论的得出和执行交易的顺序无关。也就是说一般情况下并非需要调整“交易的排列组合”来影响最终的哈希计算值结果符合“阈值”的标准。这点和比特币是完全不一样的。在交易序列唯一确定后，以太坊共识算法通过调整8字节（64bit）的nonce值来让计算最终结果符合“阈值”的要求。

ETHASH的算法更加依赖于CPU+内存的双料资源组合，比特币的SHA256算法仅仅依赖于CPU的资源，两者在挖矿公平性上也是不同的，这点在后文会有讲解。


ETHASH的设计目标
---------------------------

在比特币诞生之后经历了很长一段时间，开源社区发现仅依赖于CPU的计算的算法造成挖矿成功率向算力高的一方倾斜，并诞生出了诸如矿机这种专门的设备用于挖矿。矿机并不是一台完整的计算机，而是利用如ASIC集成电路、GPU、FPGA等专用电路进行并发操作。获得更高的多挖矿效率。这个趋势渐渐将普通人的电脑淘汰出挖矿的队伍，让话语权集中于数个矿场主手中。有违背于区块链公链“人人参与，去中心化”的精神，对区块链的分布式安全也是严重的威胁。在后世的算法设计中，就将装备了“慢CPU”的设备也能参与进挖矿定为一个重要的设计目标。PoW挖矿算法更多地让整台计算机资源都充分被利用，例如GPU、内存等设备，将单纯提高CPU的优势抹平。

ETHASH并非一日铸成，前身经历了两次演进/组合，前身称之为Dagger-Hashimoto算法。Dagger与Hashimoto是两个不同的算法，它们共同组合起来形成了一套“Memory Hard Function”也就是对内存要求较高的计算/验证算法。它的核心点就是要找到一个8字节（64bit）的nonce值。而这个随机数的产生，没有比枚举更好的策略，这样对于每个挖矿者而言，找到随机数的概率是公平的。而这个随机数的找到的概率，取决于预先设置的挖矿难度。难度越高，找到随机数的概率越小，则同样的挖矿者需要更长时间来搜寻。这让以太坊有通过设置挖矿难度动态调整出块时间的可能。我们现在习惯的15秒出一个以太坊的块，就是动态难度调整和世界千万挖矿者算力博弈的最终动态平衡。


ETHASH的挖矿运行总流程
----------------------------------

ETHASH改良了Dagger-Hashimoto，正确的说已经脱胎换骨，和原来的算法已经面目全非。它的总算法分为DAG的生成和DAG的使用（挖矿）两个部分。它有效地解决了单纯内存依赖的算法诸如Scrypt算法加密难与解密同样难的困境，也突破Dagger算法不抵抗内存共享硬件加速的困境，从全区块链数据的生成改为固定的1GB的数据的生成，支持了客户端预生成数据，保障挖矿难度的平滑过度。

总的工作量/工作量证明总体流程如下。

  - 计算一个种子（Seed），该种子的计算依赖于当本块及本块之前的所有块。
  - 从种子中计算得出一个缓存（Cache），该缓存仅仅由种子得出，是一个16MB大小的数据集。轻客户端应存储下该缓存用于日后验证。
  - 从缓存中生成一个1GB大小的数据集（DataSet），这个数据集通常称为DAG。完整客户端或者挖矿客户端需要保存该1GB大小的数据，该数据随时间线性增长。
  - 挖矿过程即为哈希过程。哈希的输入是取得1GB数据集的数个子部分，并将它们放在一起执行哈希。验证过程是可以在轻客户端通过Cache缓存生成被挖矿制定的数据碎片并执行哈希验证。故而轻客户端并不需要时刻保存1GB的DAG。
  - 每当30000个以太坊区块被发掘后，1GB的数据集将被更新一次，所以矿工的主要精力放在读取这个数据集上，而非产生数据集。为了平滑过度30000个时间点上的DAG产生带来的延迟，可以预先生成DAG数据。


ETHASH算法源代码解读
-------------------------------

我们之前探讨的算法都是纸上谈兵。接下去我们将分段用Go语言源码 方式来查看一下究竟ETHASH的挖矿和验证过程是如何进行的。探讨挖矿过程就如同探索爱丽丝仙境中的兔子洞，必须层层向下探索，它的最终解答的问题是：如何才能用ETHASH算法找到一个合法的nonce，进而形成一个块？

整个打包块的过程源于sealer（封装）函数，它需要负责封装一个合法的块，这个块的获得将是一个mine（挖矿）过程，我们来查看一下它们的函数签名。

.. code-block:: go
   :caption: /consensus/ethash/sealer.go

   func (ethash *Ethash) Seal(chain consensus.ChainReader, block *types.Block, results chan<- *types.Block, stop <-chan struct{}) error {

Seal函数总目标是找到一个nonce，让其符合块难度设定，这样块就能成为合法的块而被其他节点所接受。接受多个参数输入，并返回一个Ethash的指针。输入参数中包含了ChainReader，可料想到对于区块链前置的数据是有扫描的；Block可见最终也会修改块的部分数据，例如头部数据。我们来查看一下Ethash数据结构是怎样的：

.. code-block:: go
   :caption: /consensus/ethash/ethash.go

   type Ethash struct {
       config Config
   
       caches   *lru // LRU类型的Cache缓存避免大量反复查询
       datasets *lru // LRU类型的 DataSet数据集，防止反复生成
   
       // 挖矿相关
       rand     *rand.Rand    // 产生nonce的随机数
       threads  int           // 挖矿需要的线程数量
       update   chan struct{}
       hashrate metrics.Meter // 跟踪当下hash rate挖矿效率的参数
   
       // 远程打包者所关注的参数
       workCh       chan *sealTask   // 通知渠道，通知远程打包者打包数据
   fetchWorkCh  chan *sealWork   // 获取渠道， 远程打包者获取元数据
   submitWorkCh chan *mineResult // 提交渠道，远程打包者提交打包结果
       fetchRateCh  chan chan uint64 // 获取渠道，获取远程打包者提交挖矿效率参数值
       submitRateCh chan *hashrate   // 提交渠道，远程大包着提交挖矿效率参数值
   
       // 用于测试的钩子
       shared    *Ethash       
       fakeFail  uint64        
       fakeDelay time.Duration 
       lock      sync.Mutex      
       closeOnce sync.Once       
       exitCh    chan chan error 
   }

我们看到了熟悉的三个元素，caches、datasets、rand，这三个参数在之前的算法粗略讲解里面已经提到，是ETHASH的所使用到参数的基石，我们看下这个具体的封装区块的过程。

.. code-block:: go
   :caption: /consensus/ethash/sealer.go

   // Seal函数主要功能是触发miner函数进行挖矿操作
   func (ethash *Ethash) Seal(chain consensus.ChainReader, block *types.Block, results chan<- *types.Block, stop <-chan struct{}) error {
       // 为了简洁，我们删去了测试代码
   // 为了简洁，我们删去了共享挖矿的代码
   
       // 取消挖矿信号量
       abort := make(chan struct{})
   
       ethash.lock.Lock()
       threads := ethash.threads
       if ethash.rand == nil { // 产生一个可靠的随机数
           seed, err := crand.Int(crand.Reader, big.NewInt(math.MaxInt64))
           if err != nil {
               ethash.lock.Unlock()
               return err
           }
           ethash.rand = rand.New(rand.NewSource(seed.Int64()))
       }
       ethash.lock.Unlock()
       if threads == 0 {
           threads = runtime.NumCPU()// 决定多少并发挖矿线程
       }
       if threads < 0 {
           threads = 0 // 取消本地挖矿功能
       }
       // 将挖矿过程推送给远程挖矿者
       if ethash.workCh != nil {
           ethash.workCh <- &sealTask{block: block, results: results}
       }
       var (
           pend   sync.WaitGroup
           locals = make(chan *types.Block)
       )
       for i := 0; i < threads; i++ {
           pend.Add(1)
           go func(id int, nonce uint64) { // 最重要的函数，etash.mine为挖矿函数
               defer pend.Done()
               ethash.mine(block, id, nonce, abort, locals)
           }(i, uint64(ethash.rand.Int63()))
       }
       // 开启多线程，直到挖矿成功或者取消挖矿
       go func() {
           var result *types.Block
           select {
           case <-stop:
               // 外部取消信号捕捉，直接关闭挖矿线程
               close(abort)
           case result = <-locals:
               // 某一线程找到了合法块，通知其他挖矿线程关闭
               select {
               case results <- result:
               default:
                   log.Warn("Sealing result is not read by miner", "mode", "local", "sealhash", ethash.SealHash(block.Header()))
               }
               close(abort)
           case <-ethash.update:
               // 重启所有挖矿线程
               close(abort)
               if err := ethash.Seal(chain, block, results, stop); err != nil {
                   log.Error("Failed to restart sealing after update", "err", err)
               }
           }
           // Wait for all miners to terminate and return the block
           pend.Wait()
       }()
       return nil
   }

我们看到上文中大部分代码都在处理多线程关系，还有挖矿的开启和停止，用到了Go语言的核心功能--多线程并发。挖矿的代码就一行，是一个调用，调用了ethash.mine(block, id, nonce) 这个子函数进行真的苦力活PoW算法，下面我们查看一下这个具体挖矿的实现代码。


.. code-block:: go
   :caption: /consensus/ethash/sealer.go

   // 通过PoW挖矿行为找到最终符合条件的nonce值
   func (ethash *Ethash) mine(block *types.Block, id int, seed uint64, abort chan struct{}, found chan *types.Block) {
       // 从区块头部获取必要的信息
       var (
           header  = block.Header()
           hash    = ethash.SealHash(header).Bytes()
           target  = new(big.Int).Div(two256, header.Difficulty)
           number  = header.Number.Uint64()
           dataset = ethash.dataset(number, false)
       )
       // 开启反复试算nonce值，直到算出，或者取消。
       var (
           attempts = int64(0)
           nonce    = seed
       )
       logger := log.New("miner", id)
       logger.Trace("Started ethash search for new nonces", "seed", seed)
   search:
       for {  // 死循环开始
           select {
           case <-abort:
               // 如果我们取消了，停止
               logger.Trace("Ethash nonce search aborted", "attempts", nonce-seed)
               ethash.hashrate.Mark(attempts)
               break search
   
           default:
               // 记录尝试次数
               attempts++
               if (attempts % (1 << 15)) == 0 {
                   ethash.hashrate.Mark(attempts)
                   attempts = 0
               }
               // 重要！ 试算PoW算式下的nonce值
   digest, result := hashimotoFull(dataset.dataset, hash, nonce)
   // 重要！ 成功找到nonce值的判定标准！
               if new(big.Int).SetBytes(result).Cmp(target) <= 0 { 
                   // 更新header，试算结束！
                   header = types.CopyHeader(header)
                   header.Nonce = types.EncodeNonce(nonce)
                   header.MixDigest = common.BytesToHash(digest)
   
                   // 成功找到块打包结束
                   select {
                   case found <- block.WithSeal(header):
                       logger.Trace("Ethash nonce found and reported", "attempts", nonce-seed, "nonce", nonce)
                   case <-abort:
                       logger.Trace("Ethash nonce found but discarded", "attempts", nonce-seed, "nonce", nonce)
                   }
                   break search
               }
               nonce++
           }
       }
   
       runtime.KeepAlive(dataset)
   }

在上方的代码中，总逻辑是一个死循环，循环内部反复试算nonce值直到符合难度规定，那么判断的条件就是一个依据，如下面这行代码所示。

.. code-block:: go

   if new(big.Int).SetBytes(result).Cmp(target) <= 0 {

这句话至关重要，如果试算出来的结果result小于target，则试算成功。那么又是如何试算的呢？我们仔细看可以看到这么一个试算函数。

.. code-block:: go

   digest, result := hashimotoFull(dataset.dataset, hash, nonce)

.. code-block:: go
   :caption: /consensus/ethash/algorithm.go

   func hashimotoFull(dataset []uint32, hash []byte, nonce uint64) ([]byte, []byte) {
       lookup := func(index uint32) []uint32 {
           offset := index * hashWords
           return dataset[offset : offset+hashWords]
   }
   // 将具体的hashimoto算法触发，获得最终的结果
       return hashimoto(hash, nonce, uint64(len(dataset))*4, lookup)
   }

上文可以看到hashimotoFull函数将计算的过程完全代理给了hashimoto函数，本身并没有过多的数据计算和操作，我们再深入到最深的兔子洞—hashitmoto()函数来看一下。

.. code-block:: go

   // 针对某个header和nonce，hashimoto函数采撷DataSet中部分数据来进行哈希
   func hashimoto(hash []byte, nonce uint64, size uint64, lookup func(index uint32) []uint32) ([]byte, []byte) {
       // 计算理论上用的到的“行”
       rows := uint32(size / mixBytes)
   
       // 组合header+nonce 形成 64 byte 的seed
       seed := make([]byte, 40)
       copy(seed, hash)
       binary.LittleEndian.PutUint64(seed[32:], nonce)
   
       seed = crypto.Keccak512(seed)
       seedHead := binary.LittleEndian.Uint32(seed)
   
       // 用seed开始“混合”操作
       mix := make([]uint32, mixBytes/4)
       for i := 0; i < len(mix); i++ {
           mix[i] = binary.LittleEndian.Uint32(seed[i%16*4:])
       }
       // 混合入数据集DataSet中的数据
       temp := make([]uint32, len(mix))
       for i := 0; i < loopAccesses; i++ {
           parent := fnv(uint32(i)^seedHead, mix[i%len(mix)]) % rows
           for j := uint32(0); j < mixBytes/hashBytes; j++ {
               copy(temp[j*hashWords:], lookup(2*parent+j))
           }
           fnvHash(mix, temp)
       }
       // 压缩混合
       for i := 0; i < len(mix); i += 4 {
           mix[i/4] = fnv(fnv(fnv(mix[i], mix[i+1]), mix[i+2]), mix[i+3])
       }
       mix = mix[:len(mix)/4]
   
       digest := make([]byte, common.HashLength)
       for i, val := range mix {
           binary.LittleEndian.PutUint32(digest[i*4:], val)
   }
   // 返回混合的哈希值，输出
       return digest, crypto.Keccak256(append(seed, digest...))
   }

至此我们已经清晰地分析了整个试算nonce值的过程，整个过程犹如爱丽丝漫游仙境，需要层层向下探索兔子洞，按照“Seal  -> mine  -> hashimotoFull  -> hashimoto”的顺序一层层往下解读源代码，找到试算nonce的过程，一旦找到合法的nonce立即停止多线程的mine挖矿过程，返回最终结果给Seal函数，形成合法的区块头。




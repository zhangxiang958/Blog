title: 理解 Node.js Stream 模块
date: 2018-09-01 15:18:24
categories: Node.js

---
流概念是学习 Node 绕不过去的概念之一，它的底层代码也非常复杂，它能够优化对于文件或者数据处理的内存优化与流程优化，本文主要是讲述了对于 Stream 的实现与使用。
<!-- more -->
## 静态服务器的搭建
很多时候，我们需要搭建静态文件服务器或者向客户端传输静态文件，一般来说会有以下两种：这里例子中静态服务器会向服务端发送一个 400 MB 的文件(big.txt)。
### 使用 fs.readFile 搭建静态服务器
这里我们会直接使用 fs 模块的 readFile 函数来传递文件：
```
const fs = require('fs');

require('http').createServer((req, res) => {
    fs.readFile('./big.txt', (err, file) => {
        if (err) {
            res.end(err);
        }
        res.end(file);
    });
}).listen(3000);
```
以上代码就可以建立起一个简单的静态文件服务器，但是它会存在一些问题，因为 fs 读取文件是属于将文件全部内容都读取以 Buffer 格式存放在内存中，如果是小文件可能还能应对，但是如果是面对大文件，可能就比较吃力了，而且一旦并发量上来了，服务器就很可能会奔溃，所以为了服务器的稳定性与高性能，我们往往会使用 Stream 来建立服务器。
### 使用 Stream
在 Node.js 中，像 TCP socket 等等这些都是基于 Stream 的，这为我们使用 Stream 建立服务器提供了便利。我们如果使用 Stream 来重写上面的静态文件服务器：
```
const fs = require('fs');

require('http').createServer((req, res) => {
    let fileStream = fs.createReadStream('./big.txt');
    fileStream.pipe(res);
}).listen(3000);
```
可以看到，即使是很大的文件，服务器的内存占用还是相对比较稳定的，而且消耗也比较小。这是建立在 Stream 是使用生产者消费者的概念，当数据被读取消费的时候才会继续生产，而不是先一并读取再给下一步使用。
## Stream 的四种类型
在 Stream 中，有四种类型

 1. 可读流 Readable
 2. 可写流 Writable
 3. 双工读写流 Duplex
 4. 转化流 Transform

可读流与可写流应该很好理解，在 fs 模块中就可以通过 createReadStream 或者 createWriteStream 来创建这两者。
而对于 **Duplex** 流来说，它的特点在于既可读又可写，但是写入的数据与读取的数据之间是没有关联的，也就是说通过这个流写入的数据并不能通过该流本身的读取数据的方法拿到，典型的例子就是 **TCP Socket**, 它作为端的读写数据的关键，但是读与写的数据之间没有关联。
而对于 **Tranform** 流来说，它就像是特殊的 Duplex 流，它将将读与写联合在一起，写入的数据经过转化之后可以变为读取数据的源头，直接被下游使用，典型的例子就是 **zlib.createGzip** 可以为上游数据进行压缩然后下游得到的是压缩后的数据。
## 可读流的模式与状态
### Stream 可读流的两种模式
如果将 stream 类比成管道的话，那么管道就会有流动与停止的区别，stream 也是。flowing 模式代表流动模式，数据会自动读取并且会塞入到 data 事件的回调函数中进行消费。而 paused 模式是暂停流模式，如果需要读取数据需要手动调用 read 函数，并且是在可读流触发了 readable 事件之后表示流可读之后才能调用 read 函数进行数据读取。
#### 流动模式
```
const fs = require('fs');
const readStream = fs.createReadStream('./small.txt');

//判断是否处于流动模式
console.log(readStream._readableState.flowing);  // null

readStream.on('data', (chunk) => {
    console.log(chunk.toString());  // 这里因为内容是英文，如果是中文内容需要设置编码模式
});

console.log(readStream._readableState.flowing); // true
```
可以看到，在设置了 data 事件回调之后，可读流的内部状态变量 flowing 变为了 true，说明当前流处于流动模式。
```
const fs = require('fs');
const readStream = fs.createReadStream('./small.txt');

console.log(readStream._readableState.flowing); //判断是否处于流动模式, null

readStream.pipe(process.stdout);

console.log(readStream._readableState.flowing); // true
```
这段代码结果与上面类似，内部状态变为了 true，说明 pipe 方法也可以让可读流变为流动模式。
#### 暂停模式
```
const fs = require('fs');
const readStream = fs.createReadStream('./small.txt');

console.log(readStream._readableState.flowing);  // null

// 使用 once 来绑定监听者回调，一方面避免 readable 事件的多次触发？一方面避免内存泄漏
readStream.once('readable', () => {
    while(chunk = readStream.read()){
        readStream.pause();
        console.log(readStream._readableState.flowing); // false
        console.log(chunk.toString());
        readStream.resume();
        console.log(readStream._readableState.flowing); // true
    }
});

console.log(readStream._readableState.flowing); // null
```
 我们可以看到可读流的初始状态是 null, 如果没有设置 pipe 方法或者添加 data 事件的监听回调, 那么就一直都是 null, 但是如果我们使流的 pause 方法强制让流暂停读取数据, 那么内部状态值就变成 false, 如果调用 resume 方法又会让流变成流动模式.
#### 两种模式的相互转换
提及这两种模式，我们会先提及这两个模式是如何相互转换的：首先来讲流动模式怎么转化到暂停模式，我们可以借助 pause 方法或者将 data 事件的监听回调全部移除或调用 unpipe 方法消除 pipe 函数的影响就可以将流转化为暂停模式.
对于暂停模式转化到流模式的话, 需要为流添加了 data 事件的监听者或者调用了 pipe 方法，那么流就会变为流动状态.
其实根据上面的例子我们就可以看出, 流的初始状态是 null, 如果内部状态需要转变为 true, 或者说转化为流动模式,那么可以通过添加 data 事件回调, 调用 pipe, 调用 resume 方法等.而内部状态转变为 false 或者说转变为暂停模式, 可以通过移除 data 事件回调, 调用 unpipe 方法, 调用 pause 方法等.其实 stream 的两种形态就是流的两种使用方法.
或许会有人担心我们在创建了流之后才为流添加 data 事件回调, 会不会有一部分数据丢失? 实际上并不会, 流初始状态是 null, 所以不会立刻读取数据.

*注意 unpipe 函数调用如果没有传入参数,那么就会将所有之前 pipe 的流断开*
### Stream 可读流的三种状态
对于可读流, 内部存储了一个变量来记录当前流是否处于流动模式, 其值分别是 null, false, true:
```
readStream._readableState.flowing
```
对于它的值的变化在上面的两种流模式的转化已经提及了, 这里就不再累述.
## 实现流
当我们谈及流的时候, 一方面是它的使用, 另一方面就是它的实现, 因为某些场景下我们需要为功能创建独特的流.
### Readable 流的实现
```
const stream = require('stream');

class MyReadable extends stream.Readable {
    constructor(options) {
        super(options);
        this.count = 0;
    }
    
    _read(size) {
        if (this.count === 9) {
            this.push(null);
        } else {
            this.push(this.count.toString(), 'utf8');
            this.count++;
        }
    }
};
```
上面的代码会创建出一个独特的可读流, 打印出数字 0-8. 如何读取数据在上面使用的章节已经讲过了，可以通过 data 事件或者监听 readable 后调用 read 函数进行数据读取。
注意，read 函数与实现类的时候的 _read 函数是不一样的，_read 是子类的私有方法，而 read 方法是给数据的消费者调用进而拿到数据的。
在 _read 方法内部是必须调用 push 方法将数据压入到可读流的缓冲区里面供下游读取数据的，而 _read 函数中的 size 参数是可以忽略的，它代表的就是读取数据块的大小。
可读流的两个事件需要注意，一个 data 事件用于当有数据压入流的缓冲区的时候会触发回调函数接受数据，一个是 end 事件是在可读流接受到 EOF 信号即数据结束时触发。
### Writable 流的实现
```
const stream = require('stream');

class MyWritable extends stream.Writable {
    constructor(options) {
        super(options);
    }

    _write(buf, enc, next) {
        console.log(buf.toString());
        process.nextTick(next);
    }
}
```
与可读流的实现类似，实现可写流的时候需要实现一个内部方法 _write, 这个方法会在子类实例对象调用 write 的时候被调用，它接受三个参数，第一个参数就是写入的数据 buf， 第二个参数就是数据的编码模式 enc，第三个参数是一个函数，如果 _write 在函数中调用了这个 next 参数说明传入的数据已经做了处理，写入到底层了，可以继续写入数据了，而这个方法可以通过同步调用，也可以通过异步来调用。
```
const writeStream = new MyWritabler();

writeStream.write('a');
writeStream.write('b');
writeStream.end();
```
注意 write 是给调用者使用了，用于传入需要写入的数据，最后需要调用 end 方法表示已无数据传入。
对于可写流来说，有两个事件是需要注意的，一个是 drain 事件，一个是 finish 事件。finish 事件是在可写流的写入操作全部完成的时候触发，与可读流的 end 事件类似。而 drain 事件是在缓冲区被清空时，表示写入操作是允许的时候触发。

### Duplex 流的实现
对于 Duplex 流来说，可以说它相当于可读流与可写流的一个 hub， 将两者进行结合，但是写入的数据与读取的数据没有任何的联系，典型的例子就是 net.Socket。
因此，实现一个 Duplex 流就是同时实现一个 writable 流与 reabable 流：
```
const stream = require('stream');

class MyDuplex extends stream.Duplex {
    constructor(options) {
        super(options);
        this.data = [1,2,3,4,5];
    }
    
    _read(size) {
        if (!this.data.length) {
            this.push(null);
        } else {
            this.push(this.data.pop());
        }
    }
    
    _write(buf, enc, next) {
        console.log(buf.toString());
        process.nextTick(next);
    }
}
```
可以看到，实现的 Duplex 流既有 _read 方法，也有 _write 方法，所以即可读又可写。
### transform 流的实现
transform 流可以看做是一个特殊的 Duplex 流, transform 流的读取数据端与写入数据端是连通的，写入的数据经过处理之后会自动变为下游读取的数据, 典型的例子就是 zlib.createGzip 构造数据压缩流. 在实现一个 trangform 流的时候, 我们并不需要实现 _read 和 _write 方法, 而只需要实现特殊的 _transform 方法:
```
const stream = require('stream');

class MyTransform extends stream.Transform {
    constructor(options) {
        super(options);
    }
    
    _transform (buf, enc, next) {
        console.log(buf.toString());
        this.push(buf);
        process.nextTick(next);
    }
    
    _flush(next) {
        this.push(null);
        process.nextTick(next);
    }
}
```
像上面的例子, 实现一个 transform 流只需要实现内部的 _transform 方法就可以, 代码中并没有对数据做任何的操作,只是简单地将数据使用 push 方法推入到缓冲区当中, 这里和可读流 readable 是类似的, 然后异步地启动 next 方法告诉上游数据已经写入到底层, 可以继续写入数据, 这里和 write 可写流是类似的. 而 _flush 函数更像是一个结束函数, 用于将剩余的数据一并推入到内部缓冲区内.
## ObjectMode
对于流来说, 它不仅仅可以处理 Buffer, String 这样的数据类型, 还能处理复杂的对象类型, 而我们只需要在使用的时候传入一个 ObjectMode 的参数就可以开启处理对象类型数据的模式. 而在 Javascript 中, 几乎所有的数据都源自对象, 因此, 有了 ObjectMode 流就可以处理几乎所有的数据.
所谓 ObjectMode 的意思就是允许流是一个对象，而非字节. 而对于可读流与可写流来说，这个参数的含义是不同的：
对于可读流来说，它的意思是下游从可读流中读到的数据是对象类型，对于可写流来说，从上游写入的数据可以是对象类型。
```
const stream = require('stream');

let source = ['1', '2', { a: 'test', b: 'stream' }];
const read = new stream.Readable({ 
    objectMode: true,
    read: function(size) {
        if (source.length) {
            this.push(source.pop());
        } else {
            this.push(null);
        }
    }
});

read.on('data', (buf) => {
    console.log(buf);
});
// 上面的数据会原样输出，而不再是 Buffer <> 这样的形式
```
注意，上面的输出代码不能使用 read.pipe(process.stdout) 进行输出，因为 process.stdout 是一个可写流，而这个可写流并没有进行 objectMode 配置，是不可以接受对象类型的数据的，由此也可以看出对于可写流的 objectMode 配置是使流能够接受对象类型的值
```
// 直接使用上面的 read 流
const write = new stream.Writable({
    objectMode: true,
    write: (buf, enc, next) => {
        console.log(buf);
        process.nextTick(next);
    }
});

read.pipe(write);
```
在可读流与可写流来说，ObjectMode 都是默认对本流行为起作用的，但是作为包含可读流与可写流两种模式的 Duplex 流来说，其实可以单独对 objectMode 进行配置:
```
const duplex = new stream.Duplex({
    readableObjectMode: true,
    writableObjectMode: true
});
```
## 背压机制与 hightWaterMark
所谓背压机制就是防止出现上下游两端处理数据速度差异较大导致大量数据堆积在内存中导致进程崩溃的情况出现的机制。在上面的例子中，我们都没有使用背压机制，问题在于如果文件大小过大，底层的 socket 来不及读或写，那么就会出现大量的数据积压在内存中。
而背压机制的实现在 stream 中主要是依靠事件与 highWaterMark。highWaterMark 从字面意思是指高水位，而 stream 的流概念可以看作是一个现实的例子：流的两端是两个蓄水池，中间使用一根管子来将两端的水进行传输，那根管子就是 node 进程中的流，hightWaterMark 就是用来限制这个管子最多可以存储多少水量而不至于传输量过大导致水管破裂(进程崩溃).
而在平时使用的时候，highWaterMark 的默认值是 16kb， 也就是说一个流在传输数据过程中，如果瞬间数据量或者由于处理速度不当累积数据超过 16kb，那么流就会暂停流的生产端的动作，直到消费端将缓存区的数据消费完才会继续生产数据。
在流中有一些变量存储了当前的流的重要数据，看下面的例子：
```
const stream = require('stream');
const read = stream.Readable({ highWaterMark: '1mb' });
const state = read._readableState;
const buffer = state.buffer; // 一个 Buffer 数组，充当缓冲区
const length = state.length; // 注意 length 不表示 buffer 的长度，而是指 buffer 中所有数据的字节数，类似 Buffer.byteLength
```
那么，问题在于我们怎么知道缓存区的数据超过了 16kb 或者说消费端处理数据速度过慢需要暂停生产数据呢？
我们主要是依靠 stream 的内部的方法来判断的: 如果可写流的 write 方法或者可读流的 push 方法返回值为 false 的时候，说明内部的缓存的字节数大于的设定的 hightWaterMark, 此时我们应该暂停流的读取或写入操作。
一个可写流的背压简单例子：
```
const Read = fs.createReadStream('./filePath');
const Write = fs.createWriteStream('./dest');

const readData = () => {
    let buf;
    while(buf = Read.read()) {
        let continue = Write.write(buf);
        if (!continue) {
            return Write.once('drain', readData);
        }
    }
};

Read.on('readable', () => {
    readData();   
});
```
## 多路复用与多路分解
多路复用与多路分解在很多领域都会有相似的概念，但是在 node Stream 中，我们的用意在于将多个 stream 的数据通过一个 stream 传送到另一端并且由另一端进行数据的分解。这个部分最重要的在于定好协议,比如说在一个 buffer 数据中前 1 位说明是哪个 stream 的数据，然后前 N 位数据说明数据的长度，然后最好接上数据块，这样将数据就可以通过一个 stream 中传送然后在另外一端重新进行分配。
## Stream In Action
在实际的场景中，我们可能需要对某几个可读流都做同一个操作，就比如将多个文件合并到一个文件里面，那么我们会建立多个可读流，这个时候我们可能就需要合并可读流:
```
const stream = require('stream');
const fs = require('fs');

const handleStream = (read, dest) => {
    return new Promise((resolve, reject) => {
        read.on('end', () => {
            resolve();
        });
        read.on('error', (err) => {
            reject(err);
        });
        read.pipe(dest, { end: false });
    });
};

async function concat (streams, dest, callback) {
    if (!(dest instanceof stream.Writable)) dest = fs.WriteStream(dest);
    for (let s of streams) {
        if (!(s instanceof stream.Readable)) s = fs.createReadStream(s);
        await handleStream(s, dest);
    }
    callback();
}
```
测试代码如下:
```
console.time('streamConcat');
concat(['./logs/a.log', './logs/b.log'], './logs/dest.log', () => {
     console.log('all file concat!');
     console.timeEnd('streamConcat');
});
```
这样可以看到目标文件已经将两个文件合并了，而且内容是按照顺序的。
但是有些时候我们对于文件内容不在乎内容的顺序，只是希望单纯地将内容合并起来，那么我们可以发挥并行的优势:
```
const stream = require('stream');

const parallel = (...streams) => {
    if (Array.isArray(streams[0])) streams = streams[0];

    let streamInstance = new stream.Stream();
    streamInstance.writable = stream.readable = true;
    
    let streamCount = streams.length;
    
    function bindEvent(event) {
        let count = 0;
        return (item) => {
            let already = false;
            item.on(event, () => {
                if (already) {
                    return;
                }
                already = true;
                count++;
                if (count >= streamCount) {
                    streamInstance.emit(event);
                }
            });
        };
    }

    let bindEndEvent = bindEvent('end');
    let bindFinishEvent = bindEvent('finish');

    streams.forEach(item => {
        item.pipe(streamInstance, { end: false });
        bindEndEvent(item);
        bindFinishEvent(item);
    });

    streamInstance.write = (data) => {
        streamInstance.emit('data', data);
    };

    streamInstance.destory = () => {
        items.forEach(item => {
            item.destory && item.destory();
        });
    }

    return streamInstance;
};
```
上面的代码可以看到我们将多个流合并，并且将他们的数据异步并行地合并到目的流中，如果本地跑下面这段测试脚本:
```
const fs = require('fs');
const read1 = fs.createReadStream('./bigA.file');
const read2 = fs.createReadStream('./bigB.file');

const write = fs.createWriteStream('./testParallel');

console.time('streamParallel');
parallel(read1, read2).pipe(write).on('finish', () => {
    console.log('end');
    console.timeEnd('streamParallel');
});
```
并且使用本地的 diff 工具来查看就会发现与 concat 出来的文件不一样，parallel 流的文件内容是不按顺序的，而且在合并速度上也有差异，parallel 的速度会快于 concat:
![](http://img.ijarvis.cn/80ae17f460184316a4603698e2e489a9_de5c1028c479b5af00521ff2f5323c27.jpg)

![](http://img.ijarvis.cn/8195b426831849428762799bbc7347e2_4190f79d57907a8cfd4cefbd43162f2d.jpg)
如果你担心合并流的数量过多会导致一些问题，你还可以建立一个限制的并行流，比如将上面的 parallel 代码改为如下:
```
const stream = require('stream');

const parallel = (concurrency, ...streams) => {
    ... //头部代码如 parallel 代码
    
    let streamCount = streams.length;
    let running = 0;
    let index = 0;

    function bindEvent(event) {
        let count = 0;
        return (item) => {
            let already = false;
            item.on(event, () => {
                if (already) {
                    return;
                }
                already = true;
                count++;
                running--;
                addStream(streams[index]);
                if (count >= streamCount) {
                    streamInstance.emit(event);
                }
            });
        };
    }

    function addStream(item) {
        if (running >= concurrency || !item) {
            return;
        }
        item.pipe(streamInstance, { end: false });
        bindEndEvent(item);
        bindFinishEvent(item);
        running++;
        index++;
    }

    let bindEndEvent = bindEvent('end');
    let bindFinishEvent = bindEvent('finish');
    streams.forEach(item => {
        if (running >= concurrency) {
            return;
        }
        addStream(item);
    });

    ... // 剩下代码与 parallel 一致
};

// 测试代码: 限制同时执行的流数量为 1 
parallel(1, read1, read2).pipe(write).on('finish', () => {
    console.log('end');
    console.timeEnd('streamParallel');
});
```
这个时候你会发现由于设置并行流的数量为 1，所以得到的文件内容是与 concat 一致的而且执行效率是差不多的。

## 附录
### 多个 pipe 的流动速度
如果一个可读流有多个 pipe，那么这个管道是由读取数据最弱的下游来决定读取数据的速度的：
```
const stream = require('stream');
const fs = require('fs');

const read = fs.createReadStream('./1.pdf');
const write1 = fs.createWriteStream('./2.pdf');
const write2 = fs.createWriteStream('./3.pdf');
const pass = new stream.PassThrough({ highWaterMark: 0 });

read.pipe(write1).on('finish', () => { console.log('write1 done', new Date()); });
read.pipe(pass).on('finish', () => { console.log('pass is done', new Date()); });

console.log(pass.readableHighWaterMark);
console.log(pass.writableHighWaterMark);
setTimeout(() => {
    console.log(pass._readableState.buffer);
    pass.pipe(write2).on('finish', () => { console.log('write2 is done', new Date()); });
}, 10000);
```
打印的时间基本是同时的，即使设置了 10 秒之后才处理 write2 可写流。
## 参考资料

 - https://github.com/zhangxiang958/Node.js-Design-Patterns-Second-Edition/blob/master/Chapter5-Coding%20with%20Streams.md







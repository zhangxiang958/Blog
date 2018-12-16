title: 使用 request, mocha, chai 让接口更稳定
date: 2017-12-30 13:47:24
categories: 单元测试

---
对于接口而言, 稳定与准确是第一要义, 前端需要稳定不会出现不明错误而且结果数据正确的接口, 这样才能保页面渲染的正确性.
<!--more-->
废话少说, show you the code:
```
const request = require('request');
let sessionToken;

describe('CGI test', function(){
    it('GET /login', function(done){
        request({
            method: 'GET',
            data: {},
            type: 'json',
            headers: {
                postman-token: 'test'
            }
        }, function(res){
            res.code.should.be.equal(200);
            sessionToken = res;
            done();
        });
    });

    it('POST /update', function(){
        request({
            method: 'POST',
            data: {
                test: 'shawn'
            },
            headers: {
                postman-token: 'test',

            }
        }, function(done){
            res.statusCode.should.be.equal(200);
            done();
        });
    });
});
```
这里并不准备讲如果使用 reuqets(http 请求客户端), mocha(测试框架), chai(断言库)这些是如何去使用的, 而是注意力在为何要这么做上面.
写好单元测试代码有利于前端在请求接口的时候, 提升接口的稳定性, 而不是前端在请求接口的时候, 出现一堆未知的错误, 第二写好测试代码能够减少给自己挖坑的几率,
如果你的代码需要改动, 但是业务逻辑并不需要改动, 这时候测试代码能够快速地保证你的接口的正确性, 第三测试代码能够让后来者能容易看懂业务逻辑.

至于为什么使用 request 而不是 supertest, 这里是因为我并不追求形式的正确, 而是追求结果正确, request 我非常熟悉能够快速写出测试代码, 其二 supertest 的 agent 功能并不好用,
而且文档很烂.
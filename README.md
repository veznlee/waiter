# waiter
等待者模式


什么叫等待者模式？所谓等待者模式，从中国博大精深的汉字文化来讲，这里面必然少不了“等”这个事，那么，究竟是等什么呢？答案是等那些不确定时间先后的异步逻辑。举个例子，2022年北京要举行冬奥会，有很多国家的运动员都会前来参加，在这之前我们肯定会知道哪些国家会参加，但是真正到了要比赛的时候，我们得先有个入场仪式，让所有参赛国的成员都入场了，比赛才会开始。但问题是我们不知道哪个国家的队伍会先入场，我们唯一能确定的是，只有当所有参赛国家的队伍都入场了，我们才可以开始其他的工作。这里我们就可以把那些需要入场的队伍，看成是不确定时间先后的异步逻辑，也就是在我们执行下一项工作之前要等的东西。

但从一般的想法来讲，为了确认是不是所有的队伍都入场了，我们是不是需要派几个志愿者守在门口，时刻看着门口，进来一个数一个呢。这样的话，我们就得随时都要盯着门口了，原因很简单，毕竟不是每个国家都像中国这样不缺人口，有的国家参赛的人只有一个，你要不实时盯着，他一下子就过去了，这你要是漏数了一个，到了最后一看，国家数不够，你可就完蛋了，轻者说你工作不认真，重者人家说你们国家不尊重他们国家，指不定两个国家之间的导火线，就被你个无名小卒点着了。可是，我要眼睛都不眨一下的盯着，不是很累吗。但是，现在的人不那么笨了，不知道大家有没有受邀去参加过那种比较正式的会议。那种会议一般都会签到，你到了，你就把你的名字签上，等会议快要开始时，主办方只需要看一下签到情况，就知道邀请的人有没有来齐，这不是省事很多吗？于是，我们的代码就可以像下面这样啦！

```javascript
//等待对象
var Writer = function(){
    var dfd = [],//等待对象容器,When中传入的异步执行方法，实为事件对象数组
        doneArr = [],//成功回调方法容器，用于存放done中传入的成功回调方法
        failArr = [],//失败回调方法容器，用于存放fail中传入的失败回调方法
        slice = Array.prototype.slice,
        that = this;

    //监控对象类
    var Primise = function(){
        //监控成功状态
        this.resolved = false;
        //监控失败状态
        this.rejected = false;
    }
        
    //扩展对异步逻辑的监控方法，这两个方法都是因异步逻辑状态的改变而执行相应操作的
    Primise.prototype = {
        //解决成功，每次执行该方法都会有一个循环，看其他的监听对象是成功还是失败
        resolve : function(){
            //设置当前监控对象解决成功，每一个事件都有自己独立的监控对象，
            //都有自己的独立成功状态与失败状态
            this.resolved = true;
            //如果没有监控对象则取消执行
            if(!dfd.length)return;
            //遍历所有注册了的监控对象
            for (var i = dfd.length - 1; i >= 0; i--) {
                //如果有任意一个监控对象没有被解决或者解决失败则返回
                if(dfd[i] && !dfd[i].resolved || dfd[i].rejected){
                    return;
                }
                //如果已经解决则清除已解决监控对象
                dfd.splice(i,1);
            }
            //执行解决成功回调方法
            _exec(doneArr);
        },
        //解决失败
        reject : function(){
            //设置当前监控对象解决失败
            this.rejected = true;
            //如果没有监控对象则取消执行
            if(!dfd.length)return;
            //清除所有监控对象
            dfd.splice(0);
            //执行解决失败回调方法
            _exec(failArr);
        }
    }
        
    //创建监控对象
    that.Deferred = function(){
        return new Primise();
    }
    //监控异步方法 参数：监控对象，用于监测已经注册过的监控对象的异步逻辑
    that.When = function(){
        //设置监控对象
        dfd = slice.call(arguments);
        var i = dfd.length;
        for (--i ; i >= 0; i--) {
            //如果不存在监控对象，或者监控对象已经解决，或者不是监控对象
            if(!dfd[i] || dfd[i].resolved || dfd[i].rejected || !dfd[i] instanceof Primise){
                //清除当前监控对象
                dfd.splice(i,1);
            }
        }
        return that;
    }
    //解决成功回调函数添加方法，用于向对应的回调容器中添加相应回调
    that.done = function(){
        doneArr = doneArr.concat(slice.call(arguments));
        return that;
    }
    //解决失败回调函数添加方法，用于向对应的回调容器中添加相应回调
    that.fail = function(){
        failArr = failArr.concat(slice.call(arguments));
        return that;
    }
    //回调执行方法
    function _exec(arr){
        //遍历回调数组执行回调,注意，此处为了按先后顺序执行，不能用逆向循环
        for (var i = 0, len = arr.length; i < len; i++) {
            try{
                arr[i] && arr[i]();
            }catch(e){}
        }
    }
}
```

我想，注释已经加得足够仔细了，如果你还是看不懂，那么可以按照这样的思路来走一遍。首先，既然是等待者模式，我们肯定要先创建一个等待者了，不然谁来等我们啊，你说是不，落实到代码自然就是这样啦。
```
var  Writer = function(){}
```

等待的人创建好了，我们还要肯定还要为他加一些其他的东西啊。首先，既然是等待者，我总得知道我要等的是谁吧，所以，我们可定要有一个等待的数组，用于存放我们要等的人是哪些，这就是上面的dfd = []了。然后我们再思考，等的这个人如果来了，我是不是该做点什么，比如，倒杯热水给他喝。所以，很自然的我们就会想到对于每一个等的人，都可能会有一个成功后的回调，既然有回调，就要先把他放起来啊，因为等的人不止一个，所以很当然的就需要一个用于存放回调函数的数组了，这就是上面代码中的doneArr了，有成功，就有不成功，同理，我们就可以得出一个用于存放失败回调的数组failArr 了。既然都有回调函数了，那光有存放回调函数的数组还不行，我们还得用个方法把这些回调函数放到数组中去才行啊，同时，失败和成功的肯定要不一样吧，各放各的，这就是上面的that.done和that.fail了，分别用于把成功和失败的回调加入各自的数组。好了，既然回调函数都有了，我们肯定要运行这些回调函数嘛，不然拿来有什么用，占内存吗？所以，function _exec(){}的作用就是用于逐个的执行回调函数。监控对象就不多说了，自然是用于监控成功还是失败的了，具体的请看注释。

### 好了，我们来看一下怎么使用等待者模式吧。

```javascript
//运用场景模拟：假设页面中有多个异步回调，每个回调结束后都要有自己的业务处理
var waiter = new Writer();
 
//第1个异步回调，5秒后停止
var first = function(){
	//创建监听对象
	var dtd = waiter.Deferred();
	setTimeout(function(){
		console.log('first');
		//发布解决成功消息，执行解决成功回调，
		//当在执行成功回调时，同时会检查其他事件的最后状态，
		//如果其他事件都已经成功执行，则执行成功回调
		//如果有其他事件还未执行完毕，则只负责把自己的状态设置为成功，
		dtd.resolve();
	}, 500);
	//返回监听对象
	return dtd;
}();
 
//第2个异步回调，10秒后停止
var second = function(){
	//创建监听对象
	var dtd = waiter.Deferred();
	setTimeout(function(){
		console.log('second');
		//发布解决成功消息
		dtd.resolve();//
	}, 1000);
	//返回监听对象
	return dtd;
}();
 
//最后，我们用等待者对象监听两个异步回调的工作状态，并执行相应的成功回调与失败回调
waiter
.When(first,second)//把异步方法加入when当中监听
.done(function(){
//把成功回调方法加入donearr中保存，在监听的事件中，
//只要有一个事件的最终状态为失败，则整个结果为失败，成功队列中的方法不再执行
//当且仅当所有的最终结果为成功，才算成功，才会执行done中方法
	console.log('success');
},function(){
	console.log('success again');
})
.fail(function(){
//把失败回调方法加入failarr中保存，只要有一个事件的最终结果为失败，则执行失败回调方法
	console.log('fail');
},function(){//把失败回调方法加入failarr中保存
	console.log('fail again');
})
```

> 执行结果  
> first  
> second  
> success  
> success again



假如其中一个解决失败呢？


```javascript
var waiter = new Writer();

var first = function(){
	var dtd = waiter.Deferred();
	setTimeout(function(){
		console.log('first');
		dtd.resolve();
	}, 500);
	return dtd;
}();

var second = function(){
	var dtd = waiter.Deferred();
	setTimeout(function(){
		console.log('second');
		dtd.reject();
	}, 1000);
	return dtd;
}();
 
waiter
.When(first,second)
.done(function(){
	console.log('success');
},function(){
	console.log('success again');
})
.fail(function(){
	console.log('fail');
},function(){
	console.log('fail again');
})
```
>  执行结果  
>  first  
>  second  
>  fail  
>  fail again

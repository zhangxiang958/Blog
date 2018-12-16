title: 实现 JavaScript 继承的三种模式设计
date: 2018-05-16 20:44:24
categories: JavaScript
---

在这篇文章里面, 我们将会讨论三种不同的方式来实现 JavaScript 中的对象继承. 你将会看到我们使用其他语言例如 Java 中的通过让一个类继承一个可被多个子类继承的超类来继承其属性与方法的方式来实现继承.
也即是说, 在 Java 中, 继承是通过让一个类继承于其他的类, 然后创建这个类的实例对象来实现的, 但是在 JavaScript 中, 并没有类的概念, 继承是通过原型继承即让一个对象直接继承于另一个对象来实现的.

*注:本文为译文, 翻自: http://davidshariff.com/blog/javascript-inheritance-patterns/*
<!--more-->
## 伪类继承
伪类继承模式的目的是在 JavaScript 中通过模仿 Java 或者类 C 背景的语言的继承方式实现继承的模式. 换言之, 我们需要在伪类继承模式中实现类的概念, 让实例对象能够继承于一个类的属性与方法.
所谓伪类模式就是通过 constructor 构造器函数与 new 操作符将另一个 constructor 构造器的原型对象与本身的原型对象实现连接从而实现继承.
我们需要以下两个步骤来实现这个模式:
1. 创建一个构造器函数
2. 修改子类的原型对象指向父类的原型对象从而实现继承
```
/**
 * 修改 childObj 的原型对象指向为 parentObj 的原型对象
 **/
var extendObj = function(childObj, parentObj) {
    childObj.prototype = parentObj.prototype;
};

// human 基类
var Human = function() {};
// 继承属性或者方法
Human.prototype = {
    name: '',
    gender: '',
    planetOfBirth: 'Earth',
    sayGender: function () {
        alert(this.name + ' says my gender is ' + this.gender);
    },
    sayPlanet: function () {
        alert(this.name + ' was born on ' + this.planetOfBirth);
    }
};

// male 类
var Male = function (name) {
    this.gender = 'Male';
    this.name = 'David';
};
// 继承 human 类
extendObj(Male, Human);

// female 类
var Female = function (name) {
    this.name = name;
    this.gender = 'Female';
};
// 继承 human 类
extendObj(Female, Human);

// 创建实例对象
var david = new Male('David');
var jane = new Female('Jane');

david.sayGender(); // David says my gender is Male
jane.sayGender(); // Jane says my gender is Female

Male.prototype.planetOfBirth = 'Mars';
david.sayPlanet(); // David was born on Mars
jane.sayPlanet(); // Jane was born on Mars
```
和预期一样, 我们通过伪类继承模式实现了对象的继承, 然而, 我们可以看到, 这个模式存在着一些问题. 我们来看一下上面的示例代码中的最后一行的打印结果, *" Jane was born on Mars "*, 我们的本意其实是想让结果为 *" Jane was born on Earth "*,导致这个现象的原因就是我们修改 Male.prototype 对象的 planetOfBirth 属性为 *"Mars"*, 而 Famle 的原型对象与 Male 的原型对象通过上面实例代码实现的继承已经变为了同一个对象.

直接修改 Male 与 Human 原型对象的关系的问题就在于如果有很多子类继承于 Human, 万一其中有一个子类的原型对象上的属性或者方法被修改, 那么会影响到 Human 这个类还有继承于 Human 的所有子类. 原则上, 在实现继承中修改一个子类原型对象的属性不应该影响到其他继承于同一个父类的兄弟子类. 导致这个原因是因为在 JavaScript 中对象是引用传递而不是值传递, 这意味着 Human 的全部子类只要在其中一个子类的原型对象上做修改, 其他子类的原型对象都会受到影响.

*childObj.prototype = parentObj.prototype* 确实能够简单地实现继承, 但是, 如果你想要避免上面所说的问题, 你需要
修改 extendObj 函数来避免将 childObj 的原型对象直接与 parentObj 的原型对象直接连接, 而是通过一个中间对象来进行连接. 对上面的实例代码来说, 我们会创建一个空对像作为中间对象, 然后使用它来继承 Human 类的所有属性.

如果按照这种方式, 我们在每次调用 extendObj 函数来实现类的继承的时候, 都会创建一个新的空对像作为中间对象并让这个对象继承父类原型对象的所有属性, 而现在在一个子类原型对象的修改却不会影响其他继承于同一个父类对象的子类原型对象, 这样就可以解决对象的引用传递问题.

为了让你更加清晰, 下面这幅图展示了 extendObj 函数修改原型对象指向的逻辑:
![](http://davidshariff.com/blog/images/classical_inheritance.png)

现在, 如果你依照上面的步骤对 extendObj 函数进行修改之后再运行代码就会发现结果打印已变为 *" Jane was born on Earth "*:
```
/**
 * 创建一个新的构造器函数, 设置新的构造器函数的原型对象指向为 parentObj 的原型对象.
 * 然后设置 childObj 的原型对象指向为由刚刚创建的构造器函数创建出的实例对象.
 **/
var extendObj = function(childObj, parentObj) {
    var tmpObj = function () {}
    tmpObj.prototype = parentObj.prototype;
    childObj.prototype = new tmpObj();
    childObj.prototype.constructor = childObj;
};

// human 基类
var Human = function () {};
// 需要继承的属性与方法
Human.prototype = {
    name: '',
    gender: '',
    planetOfBirth: 'Earth',
    sayGender: function () {
        alert(this.name + ' says my gender is ' + this.gender);
    },
    sayPlanet: function () {
        alert(this.name + ' was born on ' + this.planetOfBirth);
    }
};

// male 类
var Male = function (name) {
    this.gender = 'Male';
    this.name = 'David';
};
// 继承自 Human
extendObj(Male, Human);

// female 类
var Female = function (name) {
    this.name = name;
    this.gender = 'Female';
};
// 继承自 Human
extendObj(Female, Human);

// 创建新的实例对象
var david = new Male('David');
var jane = new Female('Jane');

david.sayGender(); // David says my gender is Male
jane.sayGender(); // Jane says my gender is Female

Male.prototype.planetOfBirth = 'Mars';
david.sayPlanet(); // David was born on Mars
jane.sayPlanet(); // Jane was born on Earth
```

## 函数式继承
这个模式是由 Douglas Crockford 发明的, 这个模式允许一个对象继承于另一个对象, 并在这基础上对子实例对象进行属性增强. 在这个模式中, 先创建一个父类的实例对象, 然后通过修改这个实例对象上的属性来实现子类的属性增强, 最后将这个修改后的属性增强后的实例对象返回.
下面的代码与伪类模式的示例代码是同一场景, 但是是用函数式继承来写的:
```
var human = function(name) {
    var that = {};

    that.name = name || '';
    that.gender = '';
    that.planetOfBirth = 'Earth';
    that.sayGender = function () {
        alert(that.name + ' says my gender is ' + that.gender);
    };
    that.sayPlanet = function () {
        alert(that.name + ' was born on ' + that.planetOfBirth);
    };

    return that;
}

var male = function (name) {
    var that = human(name);
    that.gender = 'Male';
    return that;
}

var female = function (name) {
    var that = human(name);
    that.gender = 'Female';
    return that;
}

var david = male('David');
var jane = female('Jane');

david.sayGender(); // David says my gender is Male
jane.sayGender(); // Jane says my gender is Female

david.planetOfBirth = 'Mars';
david.sayPlanet(); // David was born on Mars
jane.sayPlanet(); // Jane was born on Earth
```

我们可以看到, 在这个模式里面, 我们不需要操作原型链, 构造函数与 new 关键字来创建实例对象, 而是通过每次调用继承函数生成一个新的父类实例对象然后对这个对象修改属性值或添加属性值来达到继承与实现子类的目的.

然而, 我们可以发现这样是有性能缺陷的, 每次我们需要实现继承的时候, 我们都需要创建一个新的父类实例对象以使用父类上的所有属性与方法, 那么即使是属于同一个子类, 每个实例对象之间都是独立的, 属性与方法没有实现复用, 这就导致 JavaScript 引擎在每次调用的时候都需要分配了新的内存空间来记录实例对象的所有数据, 因为实例对象的属性没有复用, 这样会导致性能问题.

当然, 这个模式也有好处, 就是由于使用函数来实现继承, 我们可以通过闭包来轻易地实现私有或公有属性与方法.我们来看一下下面关于两个子类 motorbike, boat 与父类 venicle 继承关系的示例代码:
```
var vehicle = function(attrs) {
    var _privateObj = {
        hasEngine: true
    },
    that = {};

    that.name = attrs.name || null;
    that.engineSize = attrs.engineSize || null;
    that.hasEngine = function () {
        alert('This ' + that.name + ' has an engine: ' + _privateObj.hasEngine);
    };

    return that;
}

var motorbike = function () {

    // private
    var _privateObj = {
        numWheels: 2
    },

    // inherit
    that = vehicle({
        name: 'Motorbike',
        engineSize: 'Small'
    });

    // public
    that.totalNumWheels = function () {
        alert('This Motobike has ' + _privateObj.numWheels + ' wheels');
    };

    that.increaseWheels = function () {
        _privateObj.numWheels++;
    };

    return that;

};

var boat = function () {

    // inherit
    that = vehicle({
        name: 'Boat',
        engineSize: 'Large'
    });

    return that;

};

myBoat = boat();
myBoat.hasEngine(); // This Boat has an engine: true
alert(myBoat.engineSize); // Large

myMotorbike = motorbike();
myMotorbike.hasEngine(); // This Motorbike has an engine: true
myMotorbike.increaseWheels();
myMotorbike.totalNumWheels(); // This Motorbike has 3 wheels
alert(myMotorbike.engineSize); // Small

myMotorbike2 = motorbike();
myMotorbike2.totalNumWheels(); // This Motorbike has 2 wheels

myMotorbike._privateObj.numWheels = 0; // undefined
myBoat.totalNumWheels(); // undefined
```
我们可以看到以上代码很容易地就能实现封装私有与公有属性与方法, _privateObj 对象不能在实例对象外部被修改, 它只能通过类的公有方法例如 increaseWheel() 这样的函数来修改值.同样地, _privateObj 对象的数据也不能在外部被读取, 只能通过类的公有的方法像 totalNumWheels 函数来读取对应的值.

## 原型继承
当然你也可以直接使用 JavaScript 中的原型继承方法来实现继承, 其实这样的方法才是最符合 JavaScript 语言的设计.在 ECMAScript 5 规范中, 你可以像下面这样轻易地实现继承:
```
var male = Object.create(human);
```
但是, 浏览器对这个方法的支持率并不理想, 如果浏览器还不支持这个方法, 我们只能自己写一个 create 方法来实现继承了:
```
(function () {
    'use strict';

    /***************************************************************
     * Helper functions for older browsers
     ***************************************************************/
    if (!Object.hasOwnProperty('create')) {
        Object.create = function (parentObj) {
            function tmpObj() {}
            tmpObj.prototype = parentObj;
            return new tmpObj();
        };
    }
    if (!Object.hasOwnProperty('defineProperties')) {
        Object.defineProperties = function (obj, props) {
            for (var prop in props) {
                Object.defineProperty(obj, prop, props[prop]);
            }
        };
    }
    /*************************************************************/

    var human = {
        name: '',
        gender: '',
        planetOfBirth: 'Earth',
        sayGender: function () {
            alert(this.name + ' says my gender is ' + this.gender);
        },
        sayPlanet: function () {
            alert(this.name + ' was born on ' + this.planetOfBirth);
        }
    };

    var male = Object.create(human, {
        gender: {value: 'Male'}
    });

    var female = Object.create(human, {
        gender: {value: 'Female'}
    });

    var david = Object.create(male, {
        name: {value: 'David'},
        planetOfBirth: {value: 'Mars'}
    });

    var jane = Object.create(female, {
        name: {value: 'Jane'}
    });

    david.sayGender(); // David says my gender is Male
    david.sayPlanet(); // David was born on Mars

    jane.sayGender(); // Jane says my gender is Female
    jane.sayPlanet(); // Jane was born on Earth
})();
```

## 结论
我们已经讨论了三种在 JavaScript 中实现继承的方式.大多数人都会选择使用原型继承, 但是我们可以看到伪类继承与函数式继承也有其用武之地.
不管你选择哪种方式都应该取决于你的实际情况, 还是那句话, "没有银弹", 所以你只需要根据你的实际情况作出最合适的方案选择就可以.





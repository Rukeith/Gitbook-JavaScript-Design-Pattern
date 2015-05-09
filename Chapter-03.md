#JavaScript Design Pattern - 函式
JavaScript 裏頭有各種不同定義函式的方式。首先是 function expressions 和 function declarations。
##背景
JavaScript 的函式有兩個主要特色，第一：函式是 JavaScript 的**第一級物件(first-class object)**，第二：函式提供了作用域。
函式也是物件，它們可以：

* 可以在執行期、程式執行的過程中動態建立
* 可以指定給變數、也可以將參考複製給其他變數，可以被擴充，而且除了少數的例外情況，也可以被刪除。
* 可以作為參數傳遞給其他函式，也可以作為其他函式的回傳值
* 可以有自己的屬性和方法

##釐清術語

	// 具名的函式表示式 named function expression
	var add = function add (a, b) {
		return a + b;
	};
	
	// 函式表示式，又稱為匿名函式 function expression, anonymous function
	var add = function (a, b) {
		return a + b;
	};
	
	其中兩者唯一的差別是，後者函式物件的 name 屬性會是空字串。前者則會是 "add"
	
	// 函式宣告式 function declaration
	function foo () {
		// 函式本體在這裡
	}
	
函式宣告式只能出現在 『program code』裡面，意思是在函式本體中，或在全域空間中。無法賦值給變數或屬性，或作為其他函式的參數。技術上可以將具名函式表示式指派給有著另一個名稱的變數 `var foo = function bar () {};`，但是較不建議此模式。

###函式的 Hoisting
函式宣告式和具名的函式表示式的行為非常接近。但這不全然是對的，因為他們的不同處就在於 hoisting 行為。所有的變數，不論定義在函式本體中的哪個位置，都會在幕後被 hoisting 到函式的最前端。這也適用於函式，讓人意想不到的是，函式宣告式不只是宣告，連定義也會一起移至最前端：

	// 反模式
	
	function foo () {
		alert('global foo');
	}
	function bar () {
		alert('global bar');
	}
	
	function hoistMe () {
		console.log(typeof foo); // "function"
		console.log(typeof bar); // "undefined"
		
		foo(); // "local foo"
		bar(); // bar is not a function
		
		// 函式宣告式：
		// 變數 'foo' 和它的實作都被提升至最前面
		function foo () {
			alert('local foo');
		}
		
		// 函式表示式：
		// 只有變數 'bar' 被提升
		// 不包括實作
		var bar = function () {
			alert('local bar');
		};
	}
	
	hoistMe();
	
##Callback Pattern
	
	function writeCode (callback) {
		// 做些事情...
		callback();
		// ...
	}
	
	function introduceBugs () {
		//	製造一些 bugs
	}
	
	writeCode(introduceBugs);
	
注意 introduceBugs() 傳遞給 writeCode() 時沒有加上括號。因為括號會執行函式，然而我們則是想要傳遞函式參考，讓 writeCode() 在適當的時機可以執行它。

範例：
	
	var findNodes = function () {
		var i = 100000,
			nodes = [],
			found;
		while (i) {
			i -= 1;
			// 複雜的邏輯
			nodes.push(found);
		}
		return nodes;
	};
	
	var hide = function (nodes) {
		var i = 0, max = nodes.length;
		for (; i < max; i += 1) {
			nodes[i].style.display = "none";
		}
	};
	
	hide(findNodes());
	
這樣的實作很沒效率，因為 hide() 必須對 findNodes() 回傳的節點陣列跑一次迴圈。如果可以避免這次迴圈，在 findNodes() 找到節點時就立刻隱藏，會更有效率。但如果將隱藏的邏輯放入 findNodes()，它就不泛用了。這時引進 callback pattern 將隱藏節點的邏輯作為一個回呼函式傳入，並委託它執行：

	var findNodes = function (callback) {
		var i = 100000,
			nodes = [],
			found;
		
		if (typeof callback !== "function") {
			callback = false;
		}
		
		while (i) {
			i -= 1;
			// 複雜的邏輯
			if (callback) {
				callback(found);
			}
			
			nodes.push(found);
		}
		return nodes;
	};
	
	var hide = function (node) {
		node.style.display = "none";
	};
	
	findNodes(hide);
	
callback 也可以是個現有的函式，或者是個匿名函式
	
	findNodes(function (node) {
		node.style.display = "block";
	});
	
##Callback 與作用域
某些情況下，回呼並非一次性的匿名函式或者全域函式，而是某個物件的方法。這種情況下，如果回呼的方法使用了 this 去參考其所屬的物件，會導致非預期的行為。

	var myapp = {};
	myapp.color = "green";
	myapp.paint = function (node) {
		node.style.color = this.color;
	};
	
	var findNodes = function (callback) {
		// ...
		if (typeof callback === "function") {
			callback(found);
		}
		// ...
	};
如果執行了 findNodes(myapp.paint)，他不會如預期般運作。因為 this.color 並未定義，this 會參考至全域物件，因為 findNodes() 是全域函式。解決方法是除了傳遞 callback 外，額外傳遞 callback 所屬的物件：
	 
	findNodes(myapp.paint, myapp);
	
	修改 findNodes() 來綁定物件
	var findNodes = function (callback, callback_obj) {
		// ...
		if (typeof callback === "function") {
			callback.call(callback_obj, found);
		}
		// ...
	};
	
另一種傳遞物件和方法作為 callback 的選擇，是將方法用字串傳遞：
	
	findNodes("paint", myapp);
	
	var findNodes = function (callback, callback_obj) {
		if (typeof callback === "string") {
			callback = callback_obj[callback];
		}
		
		// ...
		if (typeof callback === "function") {
			callback.call(callback_obj, found);
		}
		// ...
	};
	
##逾時
另一種 callback 模式是使用 timeout 方法：setTimeout() 和 setInterval()

	var thePlotThickens = function () {
		console.log('500ms later...');
	};
	setTimeout(thePlotThickens, 500);
	
thePlotThickens 在作為變數傳遞時沒有使用括號。因為並不想立刻執行，而只是想指向它。類似於 eval()

##callback function
函式是物件，所以也可以作為回傳值。函式可以回傳另一個更特殊化的函式，或根據輸入隨需要建立另一個函式。

	var setup = function () {
		alert(1);
		return function () {
			alert(2);
		};
	};
	
	var my = setup();	// alert 1
	my();	// alert 2
	
setup() 包裝了回傳函式，會產生一個 closure (閉包)。可以用這個 closure 來儲存一些 private 資料，其只能被回傳的函式存取。

	var setup = function () {
		var count = 0;
		return function () {
			return (count += 1);
		};
	};
	
	var next = setup();
	next(); // 1
	next(); // 2
	next(); // 3
	
#立即函式 immediate function pattern
一種語法，讓你可以在定義函式時立刻執行。本質上是函式表示式，由下列部分組成。

* 用函式表示式來定義函式
* 在函式最後加上一組括號，會讓函式立刻執行
* 將函式包在括號內 (如果不將函式賦值給一個變數才需要)  

		(function () {
			alert('watch out!');
		}());
提供一種作用域的*沙盒*給初始化的程式碼，這些工作只需執行一次。

###立即函式-參數

	(function (who, when) {
		console.log("I met " + who + " on " + when);
	}("Joe Black", new Date()));
一般來說，全域物件會作為參數傳遞給立即函式，讓內部可以存取而不必使用 window 物件。注意：不該傳遞太多變數給立即函式，容易造成理解的負擔

###立即函式-回傳值
立即函式的回傳值可以賦值給變數。

	var result = (function () {
		return 2 + 2;
	}());
另一種具有相同效果的實現方式，是省略掉包著函式的括號。因為將回傳值賦值給變數時不需要這些括號。

	var result = function () {
		return 2 + 2;
	}();
	
	or
	
	var result = (function () {
		return 2 + 2;
	})();
	
#立即物件初始化
還有一種方式可以避免全域作用域的污染，類似於立即函式模式。此模式使用一個物件，其帶有一個 init() 方法。當物件建立之後，就立即執行此方法。此 init 方法及負責所有初始化工作。

	({
		maxwidth: 600,
		maxheight: 400,
		gimmeMax: function () {
			return this.maxwidth + "x" + this.maxheight;
		},
		init: function () {
			console.log(this.gimmeMax());
		}
	}).init();
	
	兩種寫法都可使用
	({...}).init();
	({...}.init());
	
此模式的優點和立即函式模式一樣，當執行一次性的初始化工作時，可以保護全域命名空間。缺點是大多數的最小化工具無法有效的最小化這種模式，因為程式碼被包在物件裡。private 屬性和方法不會被重新命名為較短的名稱。

#設定值物件
傳遞許多參數給函式有很多不方便，有一種簡單的方式是讓所有參數替換成唯一一個，且讓此參數為一個物件。

	var conf = {
		username: 'batman',
		first: 'Bruce',
		last: 'Wayne'
	};
	
	addPerson(conf);
	
優點：

* 不需要記住參數和它們順序
* 可以安全地掠過選用參數
* 更容易閱讀和維護
* 更容易新增和移除參數

缺點：

* 需要知道參數的名稱
* 設定值物件的屬性名稱無法被最小化

#Curry
此部分會討論 currying 和部分函式應用(partial function application)。但要先討論函式應用(function application)。

##函式的應用
在一些程式語言中，函式並不是被 call or invoked，而是被 apply。

	var sayHi = function (who) {
		return "Hello" + (who ? ", " + who : "") + "!";
	};
	
	// 呼叫函式
	sayHi(); // Hello
	sayHi('world'); // Hello, world!
	
	// 應用函式
	sayHi.apply(null, ["hello"]); // Hello, hello!

可以看到呼叫和應用的結果都相同。apply() 需要兩個參數：第一是物件，用來綁定至函式內部的 this，第二個是參數陣列，會成為函式內可使用的 arguments 物件。如果第一個參數是 null，則 this 會指向全域物件。

	var alien = {
		sayHi: function (who) {
			return "Hello" + (who ? ", " + who : "") + "!";
		}
	};
	
	alien.sayHi('world'); // "Hello, world!"
	sayHi.apply(alien, ['humans']); // "Hello, humans!"
	
	此段程式碼就會把內部的 this 指向 alien

還有一種 call() 方法。但只接受一個參數。

	sayHi.apply(alien, ["humans"]);
	sayHi.call(alien, "humans");
	
##部分應用
某些情況下，不一定需要全部的參數。

	var add = function (x, y) {
		return x + y;
	};
	// 全應用
	add.apply(null, [5, 4]);
	
	// 部分應用
	var newadd = add.partialApply(null, [5]);
	newadd.apply(null, [4]);
	
部分應用給予我們另一個函式，而該函式可以在之後用別的參數呼叫。讓函式可以理解Ｋ並處理部分應用的過程稱為 currying

##Currying
Currying 命名來自 Haskell Curry，是一種轉換過程。

	// 一個已 Curry 化的 add()
	// 可接受只傳遞一部分的參數
	function add(x, y) {
		var oldx = x, oldy = y;
		if (typeof oldy === "undefined") {
			return function (new) {
				return oldx + newy;
			};
		}
		// 全應用
		return x + y;
	}
	
	// 測試
	typeof add(5); // function
	add(3)(4); // 7
	
	// 建立一個新函式並保存起來
	var add2000 = add(2000);
	add2000(10); // 2010
	
此例中，當第一次呼叫 add() 會對其回傳的內部函式產生一個 closure。這個 closure 會將原本傳進來的 x 和 y 存進 private 變數 oldx 和 oldy。
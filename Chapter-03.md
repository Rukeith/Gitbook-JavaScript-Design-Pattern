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
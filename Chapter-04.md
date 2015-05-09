#JavaScript Design Pattern - 物件建立模式
JavaScript 缺少其他語言的特別語法，namespace、模組、packages、private 屬性和靜態成員。在此會帶領一些替代的方式來實做這些功能。

##Namespace pattern
Namespace 可以降低全域變數的需求量，同時可以幫住避免命名衝突和過度的名稱前綴詞。為避免污染全域作用域，你應該為你的應用程式或函式庫建立一個全域物件，將所有的功能加入其中。

	var MYAPP = {};
	
	// 建構式
	MYAPP.Parent = function () {};
	MYAPP.Child = function () {};
	
	// 一個變數
	MYAPP.some_var = 1;
	// 物件容器
	MYAPP.modules = {};
	
	MYAPP.modules.module1 = {};
	MYAPP.modules.module1.data = {a: 1, b: 2};
	MYAPP.modules.module2 = {};
	
缺點：

* 每個變數和函式之前都要加上前綴，增加了需要的下載量
* 屬性名稱查詢變長
* 任何程式碼都可以變更這個物件

###泛用的命名空間函式
隨著程式的複雜度漸增，有時要增加命名空間時，會把已存在的屬性覆蓋掉。因此在命名前最吼可以先檢查是否存在。

	// 實作範例
	MYAPP.namespace('MYAPP.modules.module2');
	
	// 相當於
	var MYAPP = {
		modules: {
			module2: {}
		}
	};
	
這份實作是非破壞性的，如果已經存在的命名空間，就不會去建立
	
	var MYAPP = MYAPP || {};
	MYAPP.namespace = function (ns_string) {
		var  parts = ns_string.split('.'),
			parent = MYAPP,
			i;
		
		// 去除掉最前頭多餘的全域名稱
		if (parts[0] === 'MYAPP') {
			parts = parts.slice(1);
		}
		
		for (var i = 0; i < parts.length; i += 1) {
			// 如果屬性不存在則建立
			if (typeof parent[parts[i]] === "undefined") {
				parent[parts[i]] = {};
			}
			parent = parent[parts[i]];
		}
		return parent;
	};
	
	var module2 = MYAPP.namespace('MYAPP.modules.module2');
	module2 === MYAPP.modules.module2;
	
	// 忽略開頭
	MYAPP.namespace('modules.module51');
	
	// 很長的命名空間MYAPP.namespace('once.upon.a.time.there.was.this.long.nested.property');
	
###宣告相依性
將與你程式碼相依的模組宣告在函式或模組頂端是個好主意，意味著建立唯一一個區域變數，並指向所需的模組。

	var myFunction = function () {
		var event = YAHOO.util.Event,
			dom = YAHOO.util.Dom;
		
		// 函式接下來的部分
		// 使用變數 event 和 dom
	};
	
這是相當簡單的模式，但卻有許多好處

* 明確地宣告相依性，提示使用者必須確定某個特殊的 script 必須引用
* 在函式頂端的前端宣告，可以更容易找到並解決相依性
* 使用區域變數比使用全域變數來的快，效能更好
* 最小化工具會重新命名區域變數，但不會更改全域變數名稱

##Private
JavaScript 所有的物件都是 public，不像 Java 或其他語言可以表示 public、protected 和 private 的屬性和方法。

	var myObj = {
		myprop: 1,
		getProp: function () {
			return this.myprop;
		}
	};
	
	console.log(myobj.myprop);		// myprop 可被 public 存取
	console.log(myobj.getProp());	// getProp() 也是 public
	
	function Gadget() {
		this.name = 'iPad';
		this.stretch = function () {
			return 'iPad';
		};
	}
	
	var toy = new Gadget();
	console.log(toy.name);
	console.log(toy.stretch());
	
###Private 成員
雖然沒有 private 提供語法，但可以用 closure 來實作它們。這些 closure 內的變數，不會暴露於建構式外，但可以被 public 方法存取。

	function Gadget() {
		// private 成員
		var name = 'iPad';
		// public 方法
		this.getName = function () {
			return name;
		};
	}
	
	var toy = new Gadget();
	// name 會回傳 undefined
	console.log(toy.name);
	// 可以存取 name
	console.log(toy.getName()); // 'iPad'
	
其中 public 方法可以存取 private 成員，我們稱之為 privileged methods。要注意的是當從特權函式回傳一個 private 變數，這個變數是物件或陣列時，此時可以修改 private 變數，因為傳遞的是變數的參考。

###物件實字與隱私權
對於物件實字該怎麼有 private 成員，可以使用匿名函式來建立 closure

	var myobj;
	(function () {
		var name = "my, oh my";
		
		// 實作 public 的部分
		// 注意不要使用 var
		myobj = {
			getName: function () {
				return name;
			}
		};
	}());
	
	myobj.getName();	// "my, oh my"

	or
	
	var myobj = (function () {
		var name = "my, oh my";
		
		return {
			getName: function () {
				return name;
			}
		};
	}());
	
	myobj.getName();

##Module pattern
##Sandbox pattern
##靜態成員

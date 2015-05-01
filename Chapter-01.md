#JavaScript Design Pattern
---
##*基本概念*
###物件導向
JavaScript 是物件導向語言。程式碼中看到東西都可能是物件，只有五種原始型別不是：數值、字串、布林、null 和 undefined。物件是什麼？簡單的說，一個物件只是一堆具名屬性的集合，或是名值對的清單。最後一件要記住的事情是，有兩種主要的物件：  

* Native（原生物件）  
	在 ECMAScript 標準中描述。  
	*原生物件*可以被歸類為*內建物件*（例如 Array、Date）或使用者定義物件（var o = {};）。
* Host（宿主物件）
	定義在 host 環境中，例如：瀏覽器  
	例如 window 物件和所有的 DOM 物件。

###Prototype
JavaScript 沒有繼承，其實現方式就是使用原型。Prototype is object。建立的每一個函式都會自動取得一個 prototype 屬性指向一個新的空白物件。你可以為此空白物件新增會員，之後繼承此自此物件的新物件就可以把這些屬性當成自己的使用。

###嚴格模式(strict mode)
於ECMAScript 5 中定義的，可以向下相容。在每個作用域，不論是函式作用域、全域作用域或在傳入 eval() 的字串中，加入以下字串：
	
	function my () {
		"use strict";
	}

###JSLint
JavaScript 沒有靜態編譯時期檢查，所以可能因打錯字沒發現就發布一個錯誤的版本。而 JSLint 就是用來防止這件事情的。  
JSLint （http://jslint.com）是一個 JavaScript 程式碼品質工具，用來檢查你的程式碼，並對潛在的問題提出警告。在預設情況下，JSLint 會要求你的程式碼與嚴格模式相容。

##*精要*
為了避免修復 bug 期間浪費的人力，程式碼必須好維護。
容易維護的程式碼必須具備以下條件：

* 可讀性
* 一致性
* 可預料的
* 看起來像是同一人編寫
* 文件化

###減少全域變數
JavaScript 用函式管理作用域。定義函式內的變數對於該函式來說是*區域變數*。*全域變數*則是定義在函式作用域之外，或者沒有宣告就直接使用的變數。  
每種 JavaScript 執行環境都有個**全域物件**，在函式作用域之外的任何地方都可以使用 this 存取到。**所有建立的全域變數，都會成為全域物件的屬性**。瀏覽器上為方便起見，全域物件有個額外的 window 屬性，通常指向全域物件本身。

	myglobal = "hello";
	console.log(myglobal);
	console.log(window.myglobal);
	console.log(window["myglobal"]);
	console.log(this.myglobal);

使用全域變數的問題是，應用程式或網站的程式碼都共用它們，存在相同的全域命名空間(namespace)，有可能會發生命名衝突。所以全域變數用得越少越好，之後會學到減少全域變數的策略，例如 namespacing pattern，和自我執行的 immediate function。不過減少全域變數最重要的還是**永遠用 var 來宣告變數**。
JavaScript 有兩個不良特性，第一個特性，可以不經宣告就直接使用任何變數。第二個則是，JavaScript 有**隱含全域變數**的觀念，意思是任何未宣告就使用的變數都會自動成回全域物件的屬性。

	function sum (x, y) {
		// 反模式：隱含的全域變數
		result = x + y;
		return result;
	}

此程式碼裡 result 未被宣告就被直接使用。此函式呼叫一次後，在全域命名空間中多出一個變數 result，可能會造成問題。可以改善成：

	function sum (x, y) {
		var result = x + y;
		return result;
	}

另一個會建立隱含全域變數的反模式是，在單一個 var 宣告中使用連鎖賦值。例如以下程式碼中，a 是區域變數，但 b 是全域變數：

	 function foo () {
	 	var  a = b = 0;
	 }
	
但如果你事先宣告了變數，就可以使用連鎖賦值

	function foo () {
		var a, b;
		a = b = 0;
	}

>避免使用全域變數還有一個好理由：可移植性。如果你想要你的程式碼在不同的環境中執行，使用全域變數將很危險。

####遺漏「var」的副作用
隱含的全域變數和明確定義的全域變數之間有個微小的差別，就是能否用 delete 運算子將其刪除：

* 用 var 創造出的全域變數（在函式作用域外定義的）不可以刪除
* 不使用 var（不小心在函式中）隱含創造出來的全域變數可以刪除

這意思是，隱含全域變數並非真的變數，還是全域物件的屬性。屬性可以透過 delete 運算子刪除，但變數不行。

####單一 var 模式
在函式開頭使用單一 var 宣告，具有下列優點：

* 當你要尋找所有函式所需變數的時候，只需尋找單獨一個地方
* 避免未宣告就使用變數所造成的邏輯錯誤（所有宣告但未初始的變數會被初始為 undefined）
* 幫助你記得宣告變數，於是可以盡少使用全域變數
* 較少的程式碼  
	

		function func () {
			var a = 1,
				b = 2,
			 	sum = a + b,
			 	myobject = {},
			 	i,
			 	j;
		}
	
用單個 var 來宣告多個變數，並用逗號隔開。在宣告的同時**賦予初始值**是種良好的實踐方式，可以避免邏輯錯誤。在宣告時也可以做一些操作。
	
	function updateElement () {
		var el = document.getElementById("result"),
			style = el.style;
	}

####Hoisting：分散的 var 造成的問題
JavaScript 允許你在一個函式內有多個 var 宣告，並放在任何位置，而它們的行為跟在函式頂端就宣告一模一樣。這行為稱之為 **hoisting（提升）**。如果你使用了一個變數，並在之後又宣告它的話，可能造成邏輯錯誤。對 JavaScript 來說，只要變數是在相同的作用域中都會被視為已宣告，即便它在用 var 宣告之前就被使用。

	// 反模式
	myname = "global";  // 全域變數
	function func () {
		alert(myname);	// undefined
		var myname = "local";
		alert(myname);  // local
	}
	
實際的運作方式如下：

	myname = "global";
	function func () {
		var myname;
		alert(myname);
		myname = "local";
		alert(myname);
	}

>程式碼會經過兩階段的處理兩階段的處理。第一階段是語法分析和讀取內容，將產生變數、函式宣告，以及函式參數。第二階段則是執行時期，將產生函式表達式和不合格的識別字(未宣告的變數)。

###for 迴圈
使用 for 迴圈可以 iterate 整個陣列或類似陣列的物件，例如 arguments 或 HTMLCollection 物件。通常 for 迴圈的使用模式類似以下：

	for (var i = 0; i < myarray.length; i++) {
		// do something with myarray[i]
	}
	
這模式的問題在於每次都要存取陣列的長度，這會拖慢你的程式。特別是如果 myarray 不是陣列而是 HTMLCollection 物件。

HTMLCollection 是 DOM 方法回傳的一種物件，例如下面方法的回傳值：

* document.getElementsByName()
* document.getElementsByClassName()
* document.getElementsByTagName()

還有許多種 HTMLCollection 物件，它們在 DOM 標準出來之前就已經出現，直到現在仍在使用：

* document.images  
	網頁上的所有 IMG 元素
* document.links  
	所有的 A 元素
* document.forms  
	所有表單
* document.forms[0].elements  
	網頁上第一個出現的表單的所有欄位

這類集合物件的缺點是，總是立即查詢其底層的網頁。所以每次存取集合的長度時，是在直接查詢 DOM，而通常 DOM 的操作都很昂貴。因此可以改善成以下：

	for (var i = 0, max = myarray.length; i < max; i++) {
		// 操作 myarray[i]
	}
	
在重複整個 HTMLCollection 之前事先快取長度。還有可調整的是 i++ 改換成 i += 1。有兩種 for 迴圈模式的變形，引進一些細微的最佳化，特色是：

* 少用一個變數
* 遞減至0，這通常比較快，因為和 0 比較，會比和陣列長度或任何東西比較都更有效率

第一種變形模式：  
	
	var i, myarray = [];
	for (var i = myarray.length; i--;) {
		// myarray[i]
	}
	
第二種變形模式使用while迴圈：

	var myarray = [], i = myarray.length;
	while (i--) {
		// myarray[i]
	}

###for-in 迴圈
for-in 迴圈應用來重複整個非陣列物件，又稱為**列舉**(enumeration)。也可以用來列舉陣列，但不建議如此。for-in 迴圈不保證屬性列出的順序。在列舉物件屬性中很重要的一件事是：**使用 hasOwnProperty() 方法過濾掉來自 prototype chain 的屬性**。

	var man = {
		hands: 2,
		legs: 2,
		heads: 1
	};
	
	if (typeof object.prototype.clone === "undefined") {
		Object.prototype.clone = function () {};
	}
	
此範例中，使用物件實字定義了簡單的 man 物件。而在其他處，在定義 man 之前或之後，加入一個有用的 clone() 方法以擴充 Object 原型。Prototype chain 是即時的，所有的物件將自動能夠存取新加入的方法。如果要避免 clone() 方法在列舉時一併出現，需要呼叫 hasOwnProperty() 方法過濾掉來自原型的屬性。不然會有以下情況：

	// 1
	// for-in迴圈
	for (var i in man) {
		if (man.hasOwnProperty(i)) {
			console.log(i, ":", man[i]);
		}
	}
	
	console 中的結果
	hands: 2
	legs: 2
	heads: 1
	
	 // 2
	 // 反模式
	 // 不檢查 hasOwnProperty() 的 for-in 迴圈
	 for (var i in man) {
	 	console.log(i, ":", man[i]);
	 }
	 
	 console 中的結果
	 hands: 2
	 legs: 2
	 heads: 1
	 clone: function()
	 
另一種使用 hasOwnProerty() 的方法是透過 Object.prototype 呼叫：

	for (var i in man) {
		if (Object.prototype.hasOwnProperty().call(man, i)) {
			console.log(i, ":", man[i]);
		}
	}
	
此方法的優點是避免命名衝突，因為可能 man 物件複寫了 hasOwnProperty() 方法。

>**不要去擴充內建型別的原型**
>只有當下列條件的時候，你可以破例：

1. 可預測未來 ECMAScript 將實作此功能作為內建方法。
2. 確定自訂的屬性或方法並不存在。
3. 已清楚與團隊溝通過這個變更並文件化

若以上成立，可以使用以下模式加入自訂的功能Ｌ

	if (typeof Object.prototype.myMethod !== "function") {
		Object.prototype.myMethod = function () {
			// 實作功能
		};
	}
	
###Switch 模式
可以利用下面的模式提升 switch 的可讀性和穩健性

	var inspect_me = 0,
		result = '';
		
	switch (inspect_me) {
	case 0:
		result: 'zero';
		break;
	case 1:
		result = "one";
		break;
	default:
		result = "unknown";
	}
	
這個簡單的範例遵守了以下的編碼規範：

* 將每個 case 與 switch 並排（這是大括號縮排規則的例外）
* 將每個 case 內的程式碼縮排
* 用清楚的 break; 結束每個 case
* 避免未完結的 case
* 用一個 default: 結束 switch，以確保總會有個合理的結果

###避免隱含的型別轉換
JavaScript 在比較變數時會有隱含的做型別轉換。為了避免造成的困惑，應用 === 或 !== 來做比較。

###避免使用 eval() 
記住『eval() 是邪惡的』。這個函式接受任何字串，並當作程式碼來執行。使用 eval() 容易造成安全問題。可能會執行到被竄改過的程式碼。同樣的，傳遞字串給 setInterval()、setTimeout() 和 Function() 建構式都跟使用 eval() 是大同小異，同樣要極力避免。

	setTimeout(myFunc, 1000);
	setTimeout(function () {
		myFunc(1, 2, 3);
	}, 1000);
	
	// 反模式
	setTimeout("myFunc()", 1000);
	setTimeout("myFunc(1, 2, 3)", 1000);
	
###命名慣例
####讓建構式為首字母大寫
JavaScript 沒有 class，但有使用 new 引發的建構式函式：
`var adam = new Person()`
因為建構式依然只是個函式，可以依照這命名分辨是普通函式或建構式。

	function MyConstructor () {}
	function myFunction () {}
	
####字詞的分割方式
對於建構式，可以使用 upper camel case，例如 MyConstructor()，對於一般的函式名稱則可採用 lower camel case，如 myFunction()。非函式的變數，通常也採用 lower camel case，還有一種方式是全小寫，在字詞之間用底線區隔。至於方法和屬性也都是使用 lower camel case。常數或全域變數則是使用全大寫的方式來命名。還有一種是對於 private 的成員，會在前綴或尾巴加上底線做區隔。
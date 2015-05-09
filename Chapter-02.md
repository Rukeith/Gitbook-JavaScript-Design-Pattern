#JavaScript Design Pattern-literal notation patterns

##物件實字
* 將物件用大括號包起來
* 用逗號分隔物件內的每個屬性和函式
* 用分號分隔屬性名稱和屬性值
* 當你將物件賦值給一個變數，在最後的 } 之後加上分號

##建構式創造物件
JavaScript 沒有 class，但依然有 constructor functions，使用了非常類似 Java 或其他語言中以 class 為基礎的物件建立語法。可以使用自己的建構式函式建立物件，或使用這些內建的建構式，Object()、Date()和 String()等。

	// 物件實字
	var car = {
		goes: "far"
	};
	
	// 建構式
	var car = new Object();
	car.goes = "far";
	
物件實字的優點

* 程式碼相對較短
* 強調物件僅僅是可變得雜湊表，不是 class
* 不需要作用域的判斷

**建構式的陷阱**
Object() 建構式接受一個參數，且根據該參數的值決定執行哪個建構式。

	// 一個空物件
	var o = new Object();
	console.log(o.constructor === Object);
	
	// 一個數值物件
	var o = new Object(1);
	console.log(o.constructor === Number);
	console.log(o.toFixed(2)); "1.00"
	
	// 一個 string 物件
	var o = new Object("I am a string");
	console.log(o.constructor === String);
	console.log(typeof o.substring);
	
	// 一個布林物件
	var o = new Object(true);
	console.log(o.constructor === Boolean);
	
如果傳遞給 Object() 的參數是動態的，會導致非預期的結果。因此不要使用 new Object(); 應使用物件實字替代。

##自訂建構式函式

	var Person = function (name) {
		this.name = name;
		this.say = function () {
			return "I am " + this.name;
		};
	};
	
	var adam = new Person("Adam");
	adam.say();	// I am Adam
當用 new 呼叫建構式函式，以下事情會在函式內發生：

* 建立一個空物件，參考至 this 變數，並繼承此函式的原型
* 藉由 this 參考，將屬性和方法加入到此物件
* 這個 this 所參考的物件，會在最後隱含的回傳出去

>**只要是可重複利用的成員，例如方法。都應該放在 prototype 裡**

##強制 new 的模式
建構式只是使用 new 呼叫，本質上還是函式。忘了加上 new 不會造成執行上的錯誤，但會導致非預期的行為。當忘了 new 時，建構式中的 this 會指向全域物件，如果在瀏覽器中就是指向 window。

	function Waffle () {
		this.tastes = "yummy";
	}
	
	// 一個新物件
	var good_morning = new Waffle();
	console.log(typeof good_morning); // object
	console.log(good_morning.tastes); // yummy
	
	// 反模式
	var good_morning = Waffle();
	console.log(typeof good_morning); // undefined
	console.log(good_morning.tastes); // yummy
	
為了避免這種問題，可以習慣的把成員加至 that 而不是 this

	function Waffle () {
		var that = {};
		that.tastes = "yummy";
		return that;
	}
	
	或是直接回傳
	function Waffle () {
		return {
			tastes: "yummy";
		};
	}
	
	使用 that 只是一個慣例，也可以使用 self 和 me

##JSON
JSON 是一種資料傳輸格式，只是陣列實字和物件實字的組合。

	{
		"name" : "value",
		"some" : [1, 2, 3]
	}
	
和物件實字上唯一的不同點是，屬性名稱需要用引號包起來才合法。在 JSON 中不可以使用韓是和正規表示式的實字
在程式碼裡面，使用 JSON.parse() 來存許 JSON 的物件。與其相反的方法是 JSON.stringify()。可將任何物件或陣列序列化成一個 JSON 字串。
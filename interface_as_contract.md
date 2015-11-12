#Interface As Contract 介面合約

## Strong Typing & Water Fowl In (強型別 & 鴨子型別)
在前幾章，我們討論了基本的依賴注入，什麼是依賴注入；如何實現依賴注入；以及依賴注入有什麼好處，在前幾章的範例中我們也示範了將介面注入到類別中，所以，在我們繼續往下走之前，需要再深入瞭解介面，這部分也是許多 PHP 開發者所不熟悉的。

在我成為 PHP 開發者之前，我是寫 .NET 的，難道我是受虐狂嗎？在 .NET 裡介面無所不在，事實上，許多介面已經定義在 .NET 框架核心中了，其原因在於：許多 .NET 的開發語言都是強型別，例如：C#、VB.NET。
一般而言，在給函數傳值的時候，你需要指定物件的型別，或是傳入原生物件到此方法，例如，考慮以下 C#  方法：

```
public int BillUser(User user){	this.biller.bill(user.GetId(), this.amount)}
```

注意這裡，我們不僅要指定傳入參數是什麼型別，還要指定此方法的回傳值是什麼型別。 C# 重視型別的安全，除了指定的 User 物件之外，這裡不允許我們傳入其他類型的物件到 BillUser 方法中。

然而，PHP 是一種「鴨子型別」的語言。在「鴨子型別」的語言中，物件可用的方法取決於使用的方式，而非這個方法從哪裡繼承或實作而來。來看個範例：

```
public function billUser($user)
{
    $this->biller->bill($user->getId(), $this->amount);
}
```

在 PHP 裡，我們不需要在方法中指定可以接受哪種型別的參數。事實上，我們可以傳入任何型別的物件，只要這個物件能夠調用 getId 方法即可。這裡有個關「鴨子型別」的解釋：如果一個西東看起來像鴨子，叫聲也像鴨子，那它就是隻鴨子。換言之，在這個範例中，一個物件看起來是個 User，行為也像個 User，那它就是個 User。

不過，PHP 是否有強型別的功能呢？當然有！ PHP 混合了強型別和若型別的結構。為了說明這一點，我們來重寫一下 billUser 方法：

```
public function billUser(User $user)
{
    $this->biller->bill($user->getId(), $amount);
}
```

給方法加上 User 這個類型提示後，我們可以確保所有傳入 billUser 方法的參數，都是 User 的實體或是繼承自 User 的實體。
強型別與若型別各有優缺點，在強型別語言中，編譯器通常可以提供編譯時錯誤檢查的功能，這是非常有用的，方法的輸入和輸出在強型別中也是相當清楚的。

同時，強型別的特性也會使得程式沒有彈性，例如，在 Eloquent ORM 中，類似 whereEmailOrName 的動態方法就無法在 C# 這類的強型別語言中實現。
我們避免去討論強型別、弱型別哪一種比較好，而是要記住他們個別的優缺點。在 PHP 裡使用強型別特性並沒有錯，使用弱型別也沒有不好。錯誤的是，固執地只使用特定一種型別特性，而沒有針對你要解決的實際情況去思考。

## A Contract Example 介面的範例

介面就是合約，介面內不包含實作的程式碼，僅僅定義了物件必須要實作哪些方法。
當物件實作了某個介面，我們可以確保介面所定義的這些方法都能在此物件上使用。
由於合約保證一定有實作這些特定方法，藉由多型也能使強型別的語言變得更加靈活。

>Polywhat？ 多什麼型？
>
>多型的含義相當廣泛，基本上來說就是一個實體擁有多種形式，在本書中，我們講多型是一個介面有多種實作。例如，UserRepositoryInterface 可以有 MySQL 和 Redis 兩種實作，這兩者都可以代表 UserRepositoryInterface 的一個實例。

為了說明介面帶給強型別語言的靈活性，我們來寫一個飯店訂房的程式，考慮以下介面：

```
interface ProviderInterface{
    public function getLowestPrice($location);
    public function book($location);
}
```

當使用者訂房時，我們需要記錄到我們系統中，所以在 User 類別中寫些方法：

```
class User{
    public function bookLocation(ProviderInterface $provider, $location)
    {
        $amountCharged = $provider->book($location);
        $this->logBookedLocation($location, $amountCharged);
    }
```

由於我們指定了 ProviderInterface 型別提示，該 User 類別就可以安心的調用 book 方法，這使得 bookLocation 方法更加有彈性可以被複用，使用者可以任意更換其他家飯店而不影響。最後讓我們寫些程式碼來加強它的靈活性。

```
$location = 'Hilton, Dallas';

$cheapestProvider = $this->findCheapest($location, array(
    new PricelineProvider,
    new OrbitzProvider,
));

$user->bookLocation($cheapestProvider, $location);
```

太好了！不管哪一家是最便宜的，我們都能夠將它傳入 User 物件來預定房間。因為 User 物件只需要有一個符合 ProviderInterface 合約的實作就可以進行預訂。即使未來加入更多新的飯店供應商，我們的程式碼也可以運行順暢。

>Forget The Details 忘掉細節吧
>
>記住，介面實際上不做任何事，僅簡單定義了類別必須要實作的一些方法。


##Interfaces & Team Development 介面 & 團隊開發
當你的團隊在開發大型應用程式時，各個組件的開發速度會不一樣，例如有一位開發人員在處理資料層，另一位開發人員在做前端和網站/控制器層。前端開發者想要測試他的控制器，但是後端開發比較慢無法進行同步測試。如果這兩位開發者能夠先達成一份有共識的介面，後端開發的類別都遵守他，就像這樣：

```
interface OrderRepositoryInterface {
    public function getMostRecent(User $user);
}
```

一旦建立了介面，就算還沒有具體實作，前端人員也可以開始測試他的控制器了！
這樣可以讓各個組件可以有不同的開發速度，也可以方便進行單元測試。此外，這樣還可以讓組件內部的修改不會影響到其他不相關的組件，要記住，無知是一種幸福，我們寫的這些類別不需要知道其他類別是如何實作的，只要知道他們能夠實作些什麼就好。現在我們已經有一份合約了，接下來讓我們來寫控制器：

```
class OrderController {
    public function __construct(OrderRepositoryInterface $orders)
    {
        $this->orders = $orders;
    }
    public function getRecent()
    {
        $recent = $this->orders->getMostRecent(Auth::user());
        return View::make('orders.recent', compact('recent'));
    }
}
```

前端開發者甚至可以為此介面寫個「假的」實作，然後應用程式的頁面就可以被這些假數據給填充了：

```
class DummyOrderRepository implements OrderRepositoryInterface {
    public function getMostRecent(User $user)
    {
        return array('Order 1', 'Order 2', 'Order 3');
    }
}
```

一旦「假的」實作寫好了，就可以被綁定到 IoC 容器裡，然後整個應用程式都可以使用它了：

```
App::bind('OrderRepositoryInterface', 'DummyOrderRepository');
```

然而當後端開發者完成了真正的實作，像是 RedisOrderRepository。那麼 IoC 容器就可以輕易地切換到真正的實作上，整個應用程式就會使用從 Redis 讀出來的資料。

>Interface As Schematic 介面就是大綱
>
介面在開發程式的「骨架」時非常有用，使用介面來進行討論和設計都是有幫助的。例如定義一個 BillingNotifierInterface 然後討論它需要具備哪些方法，在實際動手寫代碼之前，可以使用介面來討論設計出一套好的 API。
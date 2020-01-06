# 兩站TapPay

比薪水與面試趣的付款方式中，有部分功能使用TapPay，這份文件分為幾個項目:

1. TapPay後台設定
2. 兩站API運作方式
3. 相關資料


## TapPay後台設定

#### 登入
可從TapPay官網找到登入按鈕又或是以下網址：
[https://portal.tappaysdk.com/login](https://portal.tappaysdk.com/login)

公司帳號都是固定的: `pi53906763`

:::info
:bulb:  個人帳號需請Max開通，使用預設密碼登入後可自行修改密碼
:::

![](https://i.imgur.com/u0P6IjK.png)

登入後可看見主控版畫面，由Sidebar介紹會使用到的部分

![](https://i.imgur.com/b8ny3mF.png)

#### 帳戶資訊

`TapPay SDK`需要的`Partner Key`在此取得
請編輯此頁面並將自己的公司Email加到技術窗口的部分，若是之後TapPay系統有任何通知才能夠收到信。

![](https://i.imgur.com/Cprvgmi.png)


#### 開發者

1. 應用程式:
主要在這裡取得使用`TapPay SDK`需要用到的`AppKey`和`AppID`，另外需要在`Web`的欄位設定兩站的網域。
:::info
:bulb: 圖中的 `*.pihub.dev` 是在local開發時使用的domain，可自行更換
:::

    
 ![](https://i.imgur.com/S23gBqX.png)

2. 商家管理:
分為測試與正式環境，使用TapPay時需綁定使用的刷卡銀行/刷卡方式，需要使用到商家的代號`Merchant ID`
    * 測試環境下，可直接使用這些商家來做測試
    ![](https://i.imgur.com/vr1q7sV.png)
    * 正式環境下，兩站程式碼中個別使用所需的`Merchant ID`
    ![](https://i.imgur.com/cFEhMcT.png)

3. 交易紀錄:
可以在這裡查詢測試環境與正式環境的交易紀錄
![](https://i.imgur.com/M4BckFa.png)


4. 系統設定:
這邊會需要設定公司以及線上機器的IP，才可以正常測試&使用`TapPay SDK`，會使用到系統環境設定的分頁，分為兩個部分:
    * **測試環境**
    設定為公司兩個對外IP `220.133.15.144` `114.35.222.63`
    ![測試環境](https://i.imgur.com/B7Wmx8U.png)
    * **正式環境**
    設定兩站各自的IP `interview 35.229.238.95`  `salary 34.80.193.223`
    ![正式環境](https://i.imgur.com/rf6cOqN.png)



## 兩站API運作
![](https://i.imgur.com/jwDRY6O.png)

### Route

routes/web.php

結帳頁面

```php=
<?php
// interview
Route::group(['prefix' => 'order'], function(){
    Route::get('checkout', "OrderController@checkout")->name('order.checkout');
});
// salary
Route::prefix('order')
->name('order.')
->group(function(){
    Route::get('/checkout/', [
        'uses' => 'OrderController@checkout',
        'as' => 'checkout'
    ]);
});
```

按下結帳按鈕後的Reuqest發送至此
```php=
<?php

// interview
Route::post('/order_tappay', 'OrderController@createOrderTapPay')->name('order.create.tappay');

// salary
Route::prefix('order')
->name('order.')
->middleware('auth', 'block.locked.user', 'throttle:30,1')
->group(function(){
    Route::post('tappay', [
        'uses' => 'OrderController@createOrderTapPay',
        'as' => 'create.tap'
    ]);
});
```
:::warning
:warning: 由於兩站的code只有些微差異，以下用`interview`作為例子，在`salary`都可以找到相對應的檔案
:::

### Config
1. .env
``` conf=
TAPPAY_APP_ID=15074
TAPPAY_APP_KEY="app_q3hlxsbGGtIyi7CmDGLaJOYQ7oar9r23EWl9J63i5Wy4fLcAmFqxf4PXtcM2"
TAPPAY_PARTNER_KEY="partner_1dAnB5fqva9ypMtaFgVHVXSVLfnDoijF9SGwKWzl5NzKE1vOXJK0DUMK"

TAPPAY_MERCHANT_ID="pi53906763_CTBC"
TAPPAY_SPGATEWAY_MERCHANT_ID="TAP087"
# 注意！ 此兩行會根據是 interview / salary 有所不同
# 以上透過TapPay後台取得
```

:::info
:bulb: `TAPPAY_SPGATEWAY_MERCHANT_ID`是用來讓後台查詢時使用，是藍新的商店代號，可透過TapPay後台或藍新後台取得
:::

2. config/tappay.php
```php=
<?php
return [
    /*
    |--------------------------------------------------
    | For Prime API
    |--------------------------------------------------
    |
    | 用來取得Prime時所需的設定，每項皆為必填
    |
     */
    'prime' => [
        'AppID' => env('TAPPAY_APP_ID', ''),
        'AppKey' => env('TAPPAY_APP_KEY', ''),
        'Type' => env('APP_ENV') === "production" ? "production" : "sandbox"
    ],
    /*
    |--------------------------------------------------
    | For Transaction Request
    |--------------------------------------------------
    |
    | 取得Prime後 配合訂單資訊送至tappay server進行交易時所需的設定，每項皆為必填
    |
     */
    'transaction' => [
        'PartnerKey' => env('TAPPAY_PARTNER_KEY', ''),
        'MerchantID' => env('TAPPAY_MERCHANT_ID', '')
    ],
    /*
    |--------------------------------------------------
    | For Order service
    |--------------------------------------------------
    |
    | 透過tappay交易的訂單，商店編號需使用藍新金流的商店編號，以便查帳使用
    |
     */
    'order' => [
        'MerchantID' => env('TAPPAY_SPGATEWAY_MERCHANT_ID', '')
    ],

    'url' => [
        'reserve' => env('APP_ENV') === "production" ? "https://prod.tappaysdk.com/tpc/payment/pay-by-prime" : "https://sandbox.tappaysdk.com/tpc/payment/pay-by-prime"
    ],
];
```
### FrontEnd
---
#### Controller
app/Http/Controllers/OrderController.php
```php=
<?php
public function checkout()
{
    // ...產品資訊與頁面資料 在此省略

    $TapPaySDK = [
        'AppID' => config("tappay.prime.AppID"),
        // TapPayID
        'AppKey' => config("tappay.prime.AppKey"),
        // TapPayKey
        'Type' => config("tappay.prime.Type")
        // 是 production / sandbox
    ];

    return view('order/checkout', [
        // ...帶入產品資訊與頁面資訊 在此省略
        "TapPaySDK" => $TapPaySDK
    ]);

}
```

#### View

1. blade
* resources/views/order/checkout.blade.php
```html=42
<script src="https://js.tappaysdk.com/tpdirect/v5.1.0"></script>
<script src="{{ mix('/js/checkout.js') }}"></script>
// 載入TapPay SDK & Checkout.js
```
* resources/views/components/checkout_js_variables.blade.php
```javascript=
<script>
    window.TapPaySDK = {
        AppID : {{$TapPaySDK['AppID']}},
        AppKey : {{$TapPaySDK['AppKey']}},
        Type : {{$TapPaySDK['Type']}}
    }
</script>
```
* resources/views/order/components/wrap_payment.blade.php
```html=
<div class="iw_tappay-body">
    <div class="iw_tappay-credit-info-wrap">
    <div class="iw_tappay-credit-info-detail iw_tappay-credit-info-2x">
        <label class="iw_tappay-credit-info-title">姓名</label>
        <input id="input_name" class="iw_invoice-input-item" type="text" placeholder="輸入姓名" value="{{$user['name']}}"/>
        <span class="iw_tappay-credit-info-alert">本欄位不可為空</span>
    </div>
    <div class="iw_tappay-credit-info-detail iw_tappay-credit-info-2x">
        <label class="iw_tappay-credit-info-title">手機號碼</label>
        <input id="input_cellphone" class="iw_invoice-input-item" type="text" placeholder="輸入電話" />
        <div class="iw_helper-text-wrap">
        <span class="iw_tappay-credit-info-alert">電話錯誤</span>
        <span class="iw_tappay-credit-info-extra-info">＊號碼格式：0912345678</span>
        </div>
    </div>
    <div class="iw_tappay-credit-info-detail iw_tappay-credit-info-2x iw_card-number-group">
        <label class="control-label iw_tappay-credit-info-title" for="card-number">
        <span id="cardtype"></span>信用卡卡號
        </label>
        <div class="tpfield" id="card-number"></div>
        <div class="iw_helper-text-wrap">
        <span class="iw_tappay-credit-info-alert">卡號錯誤</span>
        <span class="iw_tappay-credit-info-extra-info">支援以下卡種：VISA、Master Card、JCB</span>
        </div>
    </div>
    <div class="iw_tappay-credit-info-detail iw_tappay-credit-info-1x iw_expiration-date-group">
        <label class="control-label iw_tappay-credit-info-title">到期日(月／年)</label>
        <div class="tpfield" id="card-expiration-date"></div>
        <span class="iw_tappay-credit-info-alert">到期日錯誤</span>
    </div>
    <div class="iw_tappay-credit-info-detail iw_tappay-credit-info-1x iw_cvc-group">
        <label class="control-label iw_tappay-credit-info-title">卡片後三碼(CVV)</label>
        <div class="tpfield cvc" id="card-ccv"></div>
        <span class="iw_tappay-credit-info-alert">後三碼錯誤</span>
    </div>
    </div>
    <div class="iw_tappay-intro">面試趣採用 TapPay 金流交易系統，資料傳輸過程採用嚴密的 SSL 2048bit 加密技術保護。刷卡時直接在銀行端系統中交易，我們不會留下您的信用卡資料，以保障您的權益。</div>
</div>
```
此區塊樣板相關設定可參考TapPay Document

2. JavaScript

resources/assets/js/checkout.js
這裡相關作動都與TapPay Document中提及的一樣，以下會解釋，也可以直接到TapPay的Github查看更詳細的解說

```javascript=
/* ================  TAPPAY ================ */
function forceBlurIos() {
    if (!isIos()) {
        return
    }
    var input = document.createElement('input')
    input.setAttribute('type', 'text')
    // Insert to active element to ensure scroll lands somewhere relevant
    document.activeElement.prepend(input)
    input.focus()
    input.parentNode.removeChild(input)
}

function isIos() {
    return /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
}
// 以上是TapPay官方對於IOS裝置的一些處理

function disableSubmitBtn() {
    // disable 結帳按鈕
    submit_btn.addClass('iw_btn-disabled')
    submit_btn.prop('disabled', true)
}

function enableSubmitBtn() {
    // enable 結帳按鈕
    submit_btn.removeClass('iw_btn-disabled')
    submit_btn.prop('disabled', false)
}

function setNumberFormGroupToError(selector) {
    // 將 “正確” 樣式 套用在欄位
    $(selector).addClass('sa_is-error')
    $(selector).removeClass('sa_is-success')
}
  
function setNumberFormGroupToSuccess(selector) {
    // 將 “錯誤” 樣式 套用在欄位
    $(selector).removeClass('sa_is-error')
    $(selector).addClass('sa_is-success')
}
  
function setNumberFormGroupToNormal(selector) {
    // 恢復欄位預設樣式
    $(selector).removeClass('sa_is-error')
    $(selector).removeClass('sa_is-success')
}

TPDirect.setupSDK(window.TapPaySDK['AppID'], window.TapPaySDK['AppKey'], window.TapPaySDK['Type'])
// 帶入設置好的AppKey & Type
TPDirect.card.setup({
    fields: {
        number: {
            // css selector
            element: '#card-number',
            placeholder: '**** **** **** ****'
        },
        expirationDate: {
            // DOM object
            element: document.getElementById('card-expiration-date'),
            placeholder: 'MM / YY'
        },
        ccv: {
            element: $('#card-ccv')[0],
            placeholder: '後三碼'
        }
    },
    styles: {
      // Style all elements
      'input': {
        'color': '#5b6067'
      },
      // Styling ccv field
      'input.ccv': {
        'font-size': '16px'
      },
      // Styling expiration-date field
      'input.expiration-date': {
        'font-size': '16px'
      },
      // Styling card-number field
      'input.card-number': {
        'font-size': '16px'
      },
      // style focus state
      ':focus': {
        // 'color': 'black'
      },
      // style valid state
      '.valid': {
        'color': '#6bcba2'
      },
      // style invalid state
      '.invalid': {
        'color': '#e95658'
      },
      // Media queries
      // Note that these apply to the iframe, not the root window.
      '@media screen and (max-width: 400px)': {
        'input': {
          'color': '#5b6067'
        }
      }
    }
})

// listen for TapPay Field
TPDirect.card.onUpdate(function (update) {
    /* Change form-group style when tappay field status change */
    /* 在使用者輸入時，會自動驗證該欄位是否符合TapPay格式並更改Input樣式提醒使用者是否輸入正確 */
    /* 若沒有輸入正確則無法點擊 “結帳” 按鈕 */
    /* ======================================================= */
    // number 欄位驗證
    if (update.status.number === TapPayResponseCode.error) {
        setNumberFormGroupToError('.sa_card-number-group')
    } else if (update.status.number === TapPayResponseCode.success) {
        setNumberFormGroupToSuccess('.sa_card-number-group')
    } else {
        setNumberFormGroupToNormal('.sa_card-number-group')
    }
    // expire date 欄位驗證
    if (update.status.expiry === TapPayResponseCode.error) {
        setNumberFormGroupToError('.sa_expiration-date-group')
    } else if (update.status.expiry === TapPayResponseCode.success) {
        setNumberFormGroupToSuccess('.sa_expiration-date-group')
    } else {
        setNumberFormGroupToNormal('.sa_expiration-date-group')
    }
    // ccv 安全碼欄位驗證
    if (update.status.ccv === TapPayResponseCode.error) {
        setNumberFormGroupToError('.sa_cvc-group')
    } else if (update.status.ccv === TapPayResponseCode.success) {
        setNumberFormGroupToSuccess('.sa_cvc-group')
    } else {
        setNumberFormGroupToNormal('.sa_cvc-group')
    }
})
```
:::warning
:warning: 由於點選結帳後，作動皆相同，在此節錄TapPay作動中不同的部分
:::

一樣在`checkout.js`
```javascript=256
} else if (post_data["payment_type"] === "TP_CREDIT") {
        // fix keyboard issue in iOS device
        forceBlurIos()
    
        const tappayStatus = TPDirect.card.getTappayFieldsStatus()
        // 再次確認信用卡資訊的欄位是否符合規定
        // Check TPDirect.card.getTappayFieldsStatus().canGetPrime before TPDirect.card.getPrime
        if (tappayStatus.canGetPrime === false || tappayStatus.hasError === true) {
            makeToast('請檢查信用卡相關欄位', 2500)
            if (tappayStatus.status.number !== 0) {
                setNumberFormGroupToError('.iw_card-number-group')
            }
            if (tappayStatus.status.expiry !== 0) {
                setNumberFormGroupToError('.iw_expiration-date-group')
            }
            if (tappayStatus.status.ccv !== 0) {
                setNumberFormGroupToError('.iw_cvc-group')
            }
            enableSubmitBtn()
            return
        }
    
    
        // Get prime 取得TapPay的金鑰 prime
        TPDirect.card.getPrime(function (result) {
            // 若取得prime有問題 跳出錯誤訊息
            if (result.status !== 0) {
                alert('發生錯誤：' + result.msg)
                enableSubmitBtn()
                return
            }
            // 若取得prime成功 將prime資料帶入，送出request進行結帳
            post_data['phone'] = GetCellPhoneNumberWithFormat()
            post_data['card'] = result.card
            post_data['card_identifier'] = result.card_identifier
            post_data['clientip'] = result.clientip

            axios.post("/api/order_tappay", post_data)
            .then(function (response) {
                // 結帳成功/失敗都會回傳訂單編號，取得後redirect到訂單明細頁
                let order_number = response.data.order_number
                window.location = "/order/detail?order_no="+order_number;
            })
        })
        return
    }
```


### Backend
---
#### Controller

app/Http/Controllers/OrderController.php
:::info
:bulb: 比薪水的後端Controller在 `app/Http/Controllers/Api/OrderController.php`
:::
```php=
<?php
public function createOrderTapPay()
    {
        $products = $this->orderService->getProducts();
        $product_codes_str = implode(array_column($products, "product_code"), ",");
        /* 
            取得商品代號列表 以interview為例子:
            [subscribe_interview_90days_set,interview_badge_100]
            這兩個個別代表 interview VIP & 100積分的商品代號
         */ 
        
        // 檢查前端發來的Request內容
        \Request::validate([
            "product_code" => "required|string|in:{$product_codes_str}",
            // 商品代號，檢查是否有在上方取回的 $product_codes_str中
            "areas" => "array|size:3",
            // 如果購買的是VIP產品，前端會送來選定的三個區域
            
            "payment_type" => "required|string|in:TP_CREDIT",
            // 付款方式，檢查是否為 "TP_CREDIT"
            "phone" => "required|string",
            "name" => "required|string",
            "email" => "required|email|string",
            // 購買者電話 姓名 email

            # carrier
            "carrier_type" => "required|in:love_code,interview_account,phone_barcode_carrier,natural_person_carrier",
            "carrier_num" => "between:8,16",
            // 發票種類 & 載具號碼

            # love
            "love_code" => "in:4141,2358482,995",
            // 如果使用捐贈碼是否符合上述幾項

            # TapPay prime
            "card.prime" => "string:digits:67"
            // TapPay交易金鑰
        ]);

    
        $user = \Auth::user();
        $uid = $user->uid;
        // 取得刷卡用戶uid

        $input = \Request::input();

        $product = $this->orderService->getProduct($input["product_code"]);
        // 取得商品資訊
        
        $order_no = "R" . date('YmdHis') . str_random(3);
        // 產生訂單編號

        // 開始創立訂單與結帳流程
        $tapPayResult = $this->orderService->createTapPayOrder([
            'order_no' => $order_no,
            'uid' => $uid,
            'email' => $input["email"],
            'name' => $user["name"],
            'merchant_id' => config('tappay.order.MerchantID'),
            // 此處用的是 藍新的商店代號，方便對帳
            'is_paid' => false,
            'payment_type' => $input["payment_type"],
            // 付款方式
            'item_desc' => $product["item_desc"],
            // 商品描述
            'product_code' => $product["product_code"],
            'amount' => $product["price"],
            'message' => "等待支付中",
            'subscribed_areas' => $input["areas"] ?? null,
            // 若購買的是VIP則會有此訂閱區域的欄位
            'carrier_type' => $input["carrier_type"],
            'carrier_num' => $input["carrier_num"] ?? null,
            'love_code' => $input["love_code"] ?? null,
            // 發票資訊
            'ip' => $input["clientip"],
            'card_4_no' => $input["card"]["lastfour"],
            'card_6_no' => $input["card"]["bincode"],
            // 卡片資訊
            'tappay_info' => [
                'card_holder_name' => $input["name"],
                'prime' => $input["card"]["prime"],
                'card_info' => [
                    'bin_code' => $input["card"]["bincode"],
                    'last_four' => $input["card"]["lastfour"],
                    'country' => $input["card"]["country"],
                    'country_code' => $input["card"]["countrycode"],
                    'level' => $input["card"]["level"],
                    'type' => $input["card"]["type"],
                    'funding' => $input["card"]["funding"],
                    'issuer' => $input["card"]["issuer"]
                ],
                'card_identifier' => $input["card_identifier"]
            ]
            // TapPay Data
        ],[
            'name' => $input["name"],
            'email' => $input["email"],
            'phone' => $input["phone"]
        ]);

        if ($tapPayResult) {
            return response()->json([
                "state" => "success",
                "order_number" => $order_no,
                "message" => "成功"
            ]);
        } else {
            return response()->json([
                "state" => "error",
                "order_number" => $order_no,
                "message" => "Tap Pay 付款有異常",
            ]);
        }
    }
```

#### Service

app/Services/OrderService

```php=
<?php
public function createTapPayOrder(array $data, array $cardHolderData)
{
    $order = $this->orderRepository->create($data
    // 新增訂單
    $reserve_data = $order->toArray();
    $tapPayResponse = $this->tapPay->reserveTapPay($reserve_data, $cardHolderData);
    // 向TapPay發送結帳的Request並取得回傳
    $tapPayTradeResult = $this->updateTapPayOrder($reserve_data, $tapPayResponse);
    // 更新TapPay訂單
    
    $updatedOrder = $this->orderRepository->getByOrderNo($order["order_no"]);
    // 取得更新後的訂單資料
    
    if(!$tapPayTradeResult) {
        // 如果交易失敗，送出失敗訊息
        $errMsg = $this->tapPay->getErrorMessage($updatedOrder["tappay_info"]);
        $this->slackNotification->sendOrderFailed($updatedOrder["order_no"], $updatedOrder["item_desc"], "使用 Tap Pay", $errMsg);
        return false;
    }

    $product = $this->getProduct($order["product_code"]);
    $update_data = [
        "card_4_no" => $tapPayResponse["card_info"]["last_four"]
    ];
    // 取得商品資訊與建立update_data
    
    $this->obtainProduct($updatedOrder["uid"], $updatedOrder, $product);
    // 根據商品資訊 賦予會員相關商品權限/積分
    $this->issueInvoice($updatedOrder["uid"], $update_data , $updatedOrder, $product);
    // 開立發票
    $this->slackNotification->sendOrderPaid($updatedOrder["order_no"], $updatedOrder["item_desc"] . ", 使用 Tap Pay" ?? "", $updatedOrder["amount"] ?? "");
    // 發送Slack通知
    return true;
}

public function updateTapPayOrder(array $order, array $response)
{   
    if($response["status"] !== 0) {
        // 根據TapPay Document 如果此代碼是0表示交易失敗
        $this->orderRepository->update($order["order_no"], [
            "message" => "付款失敗",
            "tappay_info" => [
                "status" => $response["status"],
                "msg" => $response["msg"]
            ]
        ]);
        return false;
    }

    $response["card_holder_name"] = $order["tappay_info"]["card_holder_name"];

    $this->orderRepository->update($order["order_no"],[
        "is_paid" => true,
        "message" => "付款成功",
        "auth" => $response["auth_code"],
        "tappay_info" => $response,
        // 更新我們自定義的主要欄位
        "card_4_no" => $response["card_info"]["last_four"],
        "card_6_no" => $response["card_info"]["bin_code"]
    ]);

    return true;
}
```

#### Utils

app/Utils/TapPay.php

此檔案專門用來處理`TapPay`交易`Request`和取得相關錯誤代碼的`ErrorMessage`

```php=
<?php
namespace App\Utils;

use GuzzleHttp\Client;

class TapPay
{
    private $client;
    private $headers;

    const ErrorMessage = [
        0 => "成功",
        2 => "查詢結束",
        4 => "IP 不符",
        // .......... 以下省略 皆為錯誤代碼回應訊息
    ];

    public function __construct(Client $client)
    {
        $this->client = $client;
        $this->headers = [
            'Content-Type' => 'application/json; charset=UTF-8',
            'x-api-key' => config("tappay.transaction.PartnerKey")
        ];
    }

    public function reserveTapPay(array $Order, array $cardHolderData)
    {
        $url = config("tappay.url.reserve");

        $body = [
            "partner_key" => config("tappay.transaction.PartnerKey"),
            "prime" => $Order["tappay_info"]["prime"],
            "merchant_id" => config("tappay.transaction.MerchantID"),
            "amount" => $Order["amount"],
            "details" => $Order["item_desc"],
            "order_number" => $Order["order_no"],
            "currency" => "TWD",
            "cardholder" => [
                "phone_number" => $cardHolderData["phone"],
                "name" => $cardHolderData["name"],
                "email" => $cardHolderData["email"]
            ]
        ];

        $response = $this->client->post($url, [
            'json' => $body,
            'headers' => $this->headers
        ]);

        $response_body = json_decode($response->getBody()->getContents(), true);
        
        return $response_body;
    }

    public function getErrorMessage(array $tappayInfo) {
        if (!array_key_exists($tappayInfo["status"], self::ErrorMessage)) {
            return $tappayInfo["msg"];
        }
        return self::ErrorMessage[$tappayInfo["status"]];
    }
}
```

## 相關資料

* TapPay Document
    * [Front End](https://docs.tappaysdk.com/tutorial/zh/web/front.html#tappay-fields-v5-1-0)
    * [Front End Example](https://github.com/TapPay/tappay-web-example/tree/master/TapPay_Fields)
    * [Back End](https://docs.tappaysdk.com/tutorial/zh/back.html#pay-by-prime-api)
    * [Error Message](https://docs.tappaysdk.com/tutorial/zh/error.html#error)

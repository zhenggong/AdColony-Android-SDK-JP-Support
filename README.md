![AdColony Logo and Title](assets/logo-title.png)

##基本情報##
###AdColony SDK 取得URL###

* AdColony-Unity-SDK
https://github.com/AdColony/AdColony-Unity-SDK

* AdColony-Android-SDK 
https://github.com/AdColony/AdColony-Android-SDK

* AdColony-AdobeAIR-SDK
https://github.com/AdColony/AdColony-AdobeAIR-SDK

***
AdColonyはアプリケーションのあらゆる場所にHD動画広告を配信することができます。動画を再生完了した時点でユーザに仮想通貨を付与するV4VC広告も提供しています。

###注意###
* AdColony Android SDKの現在のバージョンは2.3.4です。
* 本SDKはAndroid OS 4.0(APIレベル14)から動作対象となります。
* AdColonyの広告配信を行うことができるデバイスの最小メモリ(アプリベース)は32MBです。32MBよりも小さいデバイスは再生することができません。
* アプリケーションのターゲットがAPI 18以上かつ、ProGuardを有効にしている場合、proguard-project.txtファイルに下記のコード追加してください。 `-dontwarn android.webkit.**`
* GoogleのAdvertising IDを取得するため、プロジェクトの中にGoogle Play Services 4.0+ を追加してください。追加しない場合表示できる広告の数は少なくなります。
* データ収集に関しては以下のページを参照してください。 http://support.adcolony.com/customer/portal/articles/1605954-data-and-clickstream-collection

***
###Contents###
* [Project Setup](#project-setup)
* [Showing Videos Ads](#showing-videos-ads)
    * [Showing Interstitial Videos](#showing-interstitial-videos)
    * [Showing V4VC Videos](#showing-v4vc-videos)
    * [Showing Instant Feed™ Videos](#Showing Instant Feed™ Videos)
* [よくある質問](#よくある質問)
    * [基本情報に関して](#基本情報に関して)
    * [SDK仕様に関して](#sdk仕様に関して)
    * [動画再生に関して](#動画再生に関して)
    * [ストア申請に関して](#ストア申請に関して)

##Project Setup##
下記のステップに従ってAdColony動画広告をプロジェクトに導入してください。
####Step 1: SDKファイルの導入####
* プロジェクトのlibsフォルダー内に、adcolony.jarをコピーします。
* プロジェクトのlibsフォルダー内に、armeabフォルダーをコピーします。
  * adcolony.jarとはarmeabフォルダー両方は解凍したSDKファイルの"Library"フォルダ内にあります。

***
####Step 2: AndroidManifest.xmlの修正####
"AndroidManifest.xml"に下記のパーミッションが追加されているのを確認してください。
1. INTERNET
2. ACCESS_NETWORK_STATE
3. WRITE_EXTERNAL_STORAGE (オプション)
4. VIBRATE (オプション)
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.VIBRATE" /> 
```
Dynamic End Cardのパフォーマンスを最適化するために、下記のようにハードウェアアクセラレーションを有効にしてください。
```xml
android:hardwareAccelerated="true"
```
下記、Acitivityの設定を<application>タグの間にコピーしてください。
```xml
<activity android:name="com.jirbo.adcolony.AdColonyOverlay"
android:configChanges="keyboardHidden|orientation|screenSize"
android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />

<activity android:name="com.jirbo.adcolony.AdColonyFullscreen"
android:configChanges="keyboardHidden|orientation|screenSize"
android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen" />

<activity android:name="com.jirbo.adcolony.AdColonyBrowser"
android:configChanges="keyboardHidden|orientation|screenSize"
android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen" />
```
**注意:** アプリがAPI レベル13以下に対応している場合、上記configChanges設定からscreenSizeを消してください。

***
####Step 3: Adcolonyアカウント情報の取得####

アカウント情報の取得の以下の通りになります。

1.Glossomにてapp ID、zoneIDを発行しお渡しします。
2.広告を表示するコードを実装して下さい。

##Showing Videos Ads##

##Showing Interstitial Videos##
AdColonyインタースティシャルビデオは動画に続いてエンドカードが表示される動画広告です。

###Instructions###
[[Project Setup]]を設定した後に、以下のステップでインタースティシャル広告を表示することができます。
####Step 1: Adcolonyの設定####
Adcolonyライブラリをインポートしてください。
```java
import com.jirbo.adcolony.*;
```
次に、以下のように設定メソッドを**main Activity** の onCreateメソッドの中で呼び出してください。
```java
AdColony.configure(this, client_options, app_id, zone_ids);
```
client_optionsをアプリの情報に入れ替えてください (e.g. "version:2.1,store:google" - 詳しくは [API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details) に参照してください) 。AdColonyV4VCAdを作成するときに使用したzone_idのV4VCが有効になったことを確認してください。 <br><br>
**Note:** この設定関数は**main Activity's** onCreateメソッドで **一回** 実行してください。 また、アプリケーションが実行する間このAcitivityの参照は消さないようにしてください。

===
####Step 2: PauseとResumeの設定####
アプリ中で全てのActivityのadcolonyメソッドを呼び出すために、下記のようにonPause / onResumeでオーバーライドしてください。
```java
public void onPause() 
{
  super.onPause();
  AdColony.pause(); 
}

public void onResume() 
{
  super.onResume();
  AdColony.resume( this ); 
}
```

===
####Step 3: AdColonyV4VCAdオブジェクトの作成と動画の再生####
Adcolonyのインタースティシャル動画広告を再生するには、下記のようにAdColonyVideoAdオブジェクトを作成してshowメソッドを実行してください。

```java
AdColonyVideoAd ad = new AdColonyVideoAd(zone_id);
ad.show();
```
Or:
```java
AdColonyVideoAd ad = new AdColonyVideoAd();
ad.show();
```
zone_idはAdColony.configureで設定されたIDリスト上の管理画面で取得した有効なIDです。<br><br>
**Note:** また、これは基本の実装方法です。さらに高度な設定には下記の記述と[API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details) を参照してください。

##Showing V4VC Videos##
AdColony V4VC (Videos-for-Virtual-Currency)は[[interstitial ads|Showing Interstitial Videos]]の上で実装した動画広告を再生完了した時点で、ユーザに仮想通貨を付与することができるシステムです。AdColony V4VCはユーザの仮想通貨残高を追跡しません。ユーザに仮想通貨を付与すべき時点でアプリケーションに通知する機能を提供しています。<br><br>
[Basics](#basics)<br>
[Advanced Usage](#advanced-usage)
###Basics###
####Step 1: Adcolonyの設定####
Adcolonyライブラリをインポートしてください。
```java
import com.jirbo.adcolony.*;
```
次に、以下のように設定メソッドを**main Activity** の onCreateメソッドの中で呼び出してください。
```java
AdColony.configure(this, client_options, app_id, zone_ids);
```

client_optionsをアプリ情報へ入れ替えてください (e.g. "version:2.1,store:google" - 詳しくは [API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details) に参照してください) 。また、app_id/zone_idsを管理画面上から取得して下さい [Control Panel](http://clients.adcolony.com) ( "[Setting up Apps and Zones](http://support.adcolony.com/customer/portal/articles/761987-setting-up-apps-zones)" に参照)。 AdColonyV4VCAdを作成するときに使用したzone_idのV4VCが有効であることを確認してください。 <br><br>
**Note:** この設定関数は**main Activity's** onCreateメソッドで **一度** 限り実行してください。 アプリケーションが実行する間にこのAcitivityの参照は消さないようにしてください。

===
####Step 2: PauseとResumeの設定####
アプリ中全てのActivityにadcolonyメソッドを呼び出すために、下記のようにonPause / onResumeでオーバーライドして下さい。
```java
public void onPause() 
{
  super.onPause();
  AdColony.pause(); 
}

public void onResume() 
{
  super.onResume();
  AdColony.resume( this ); 
}
```

===
####Step 3: AdColonyV4VCAdオブジェクトの作成と動画の再生####
Adcolony V4VC動画広告を再生するには、下記のようにAdColonyV4VCAdオブジェクトを作成してshowメソッドを実行してください。
```java
AdColonyV4VCAd ad = new AdColonyV4VCAd(v4vc_zone_id);
ad.show();
```
Or
```java
AdColonyV4VCAd ad = new AdColonyV4VCAd();
ad.show();
```

v4vc_zone_idはAdColony.configureで設定されてるIDリスト中にある管理画面で取得した有効なV4VC枠IDです。<br><br>
**Note:** これは基本の実装方法です。さらに高度な設定には下記の記述と[API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details)を参照してください。

===
###Advanced Usage###
[AdColonyV4VCListener](#adcolonyv4vclistener)<br>
[Pre and Post-Popups](#pre-and-post-popups)<br>
[Server-Side Rewards](#server-side-rewards)
####AdColonyV4VCListener####
V4VC動画を再生した後にAdcolonyからアプリに通知します。一つのAdColonyV4VCListenerはアプリに一度だけ追加してください。また、AdColonyV4VCListenerを継承してAdColony.configureのすぐ後に追加してください。
```java
AdColonyV4VCListener listener = new AdColonyV4VCListener()
{
  public void onAdColonyV4VCReward(AdColonyV4VCReward reward)
  {
    //Just an example, see API Details page for more information.
    if(reward.success())
    {
      //Reward user
    }
  }
};

AdColony.addV4VCListener(listener);
```
===
####Pre and Post-Popups####
下記のようにユーザに動画再生する前の確認メッセージ(Pre)及び成果通知の確認メッセージ(Post)を、事前or事後のポップアップダイアログにて表示することを設定できます。

事前の確認メッセージを表示する場合
```java
AdColonyV4VCAd ad = new AdColonyV4VCAd(v4vc_zone_id).withConfirmationDialog();
ad.show();
```
事後の確認 メッセージを表示する場合
```java
AdColonyV4VCAd ad = new AdColonyV4VCAd(v4vc_zone_id).withResultsDialog();
ad.show();
```
両方を表示する場合
```java
AdColonyV4VCAd ad = new AdColonyV4VCAd(v4vc_zone_id).withConfirmationDialog().withResultsDialog();
ad.show();
```

===
####Server-Side Rewards####
仮想通貨のセキュリティ強化のために、ハッシングメッセージを使用した開発者のサーバーへ仮想通貨を処理するコールバックを提供しています。サーバー側でゲームのユーザに仮想通貨を付与するコールバックURLを実装する必要があります。AdcolonyからこのURLに必要なパラメーターを追加して御社のサーバーにポストバックします。それを受けてユーザに仮想通貨を付与してください。 <br><br>
Adcolonyはサーバー側がなくてもクライアント側のみ仮想通貨を付与することができますが、この方法は不正を完全に防げるのが困難なため推奨致しません。システム管理上のサーバーを使用できない場合、video-ad@glossom.co.jpに問い合わせて下さい。

####Step 1####
コールバック受信のために、サーバーのURLを作成してください。このURLは認証をかけないでください。コールバックURLは、申し込み書に記載のうえGlossomにご連絡してください。

===
####Step 2####
サーバーの応答がトランザクションに基づいた適切なものであることを確認してください。URLのフォーマットは以下になります。ただし括弧内のものはアプリやトラザクションによって変わります。
```
[http://www .yourserver.com/anypath/callback_url.php]?id=[ID]&uid=[USER_ID]&zone=[ZONE_ID]&amount=[CURRENCY_AMOUNT]&currency=[CURRENCY_TYPE]&verifier=[HASH]&open_udid=[OPEN_UDID]&udid=[UDID]&odin1=[ODIN1]&mac_sha1=[MAC_SHA1]
```
URL Parameter | Type | Purpose
--- | --- | ---
id | Positive long integer | Unique V4VC transaction ID
uid | String | AdColony device ID
amount | Positive integer | Amount of currency to reward
currency | String | Name of currency to reward
open_udid | String | OpenUDID
udid | String | Apple UDID
odin1 | String | Open Device Identification Number (ODIN)
mac_sha1 | String | SHA-1 hash of lowercase colon-separated MAC address
custom_id | String | Custom user ID
verifier | String | MD5 hash for message security

Androidの場合open_udid, udid, odin1, and mac_sha1は常に空です。 アプリからカスタマイズIDを渡したい場合、zoneのコールバックURLの後ろに“&custom_id=[CUSTOM_ID]”をつけてください。このIDも文字(verifier)の最後に追加してチェックしてください。<br><br>

コールバックの実装は、MD5がサポートしている言語であるならば、PHPにかぎらず言語の指定はありません。<br><br>

参考のために、下記のコードではPHPとMySQLを使って、パラメーターの処理とMD5でのチェック、トラザクションの重複チェックを正しくレスポンス実装しました。
```php
<?php

    $MY_SECRET_KEY="Thiscomesfromadcolony.com";

    $trans_id=mysql_real_escape_string($_GET['id']); 
    $dev_id=mysql_real_escape_string($_GET['uid']); 
    $amt=mysql_real_escape_string($_GET['amount']); 
    $currency=mysql_real_escape_string($_GET['currency']); 
    $verifier=mysql_real_escape_string($_GET['verifier']);

    //verify hash 
    $test_string="".$trans_id.$dev_id.$amt.$currency.$MY_SECRET_KEY; 
    $test_result=md5($test_string);
    if($test_result!=$verifier)
    {
      echo"vc_decline";
      die; 
    }

    //check for a valid user 
    $user_id=//get your internal user id from the device id here 
    if(!$user_id)
    {
      echo"vc_decline";
      die; 
    }

    //insert the new transaction 
    $query="INSERT INTO AdColony_Transactions(id,amount,name,user_id,time)".
      "VALUES($trans_id,$amt,'$currency',$user_id,UTC_TIMESTAMP())"; 
    $result=mysql_query($query);
    if(!$result)
    {
      //check for duplicate on insertion 
      if(mysql_errno()==1062)
      {
        echo"vc_success";
        die; 
      }
      //otherwise insert failed and AdColony should retry later 
***

      else
      {
        echo"mysql error number".mysql_errno();
        die; 
      }
    }
    //award the user the appropriate amount and type of currency here 
    echo"vc_success";

?>
```
上記コードで使ったMySQLデータベースは下記のように作成できます。
```mysql
CREATE TABLE `AdColony_Transactions` (
`id` bigint(20) NOT NULL default '0',
`amount` int(11) default NULL,
`name` enum('Currency Name 1') default NULL, `user_id` int(11) default NULL,
`time` timestamp NULL default NULL,
PRIMARY KEY (`id`)
);
```
重複付与しないために、トラザクションidを必ず保存してください。毎回コールバックURLが呼ばれる時にトラザクションIDが重複してるかどうかをチェックしてください。既に処理された場合、再度ユーザへ付与する必要はなくレスポンスが成功した場合と同様の対応を行って下さい<br><br>
重複チェックの後にユーザに指定したタイプ＆数量の仮想通貨を付与してください。

===
####Step 3####
サーバーの応答がトランザクションに基づいた適切なものかの確認<br>
* "vc_success"
  * トランザクションが終了。コールバックが受信され、ユーザが入金した場合、この値を返します。
* "vc_decline" or "vc_noreward"
  * トランザクションが終了。user_idが有効でない場合か、セキュリティチェックが渡されていない場合、この値を返します。
* 上記以外
  * サーバーへの再試行。エラーがある場合に使用されます。

**Note:** ユーザに対して付与を行わなくて良い場合は、user _idが無効、セキュリティが通らない、トランザクションがすでに報告され重複した場合のみになりますので、これらの条件に当てはまらないユーザには必ず付与を行って下さい。

##Showing Instant Feed™ Videos##
###Notes###
**Instant-Feed広告を導入する興味があれば、Video-ad@glossom.co.jpにお問い合わせください。

実装と有効なインプレションに関する契約上の要件を遵守するために、パブリッシャーは下記のことを遵守する必要があります。<br><br>
1. 動画広告は必ず見えるところに配置して、その上に何も重ねないで下さい。<br>
2. 広告は自動的に再生/停止/再開する契約上の要件を遵守してください。無許可のタクティックス使ってこの動作を修正したりしないてください。

###Overview###
AdColony Instant-Feed™は広告を直接自分のアプリのコンテンツに組み込める一つの新しいネイティブ広告ユニットです。既存のユーザーエクスペリエンスとマッチすることができます。

Instant-Feed広告は二つのエクスペリエンスがあります。：一つ目は広告のテキスト、画像、ネィティブの動画とエンゲージメントを含めるコンテナーを自分のビューに組み込むことができます。二つ目はユーザーが動画コンテナーをタップした時にAdcolony SDKが自動でフルスクリーンの動画を再生することができます。
<div align="center">
<img src="assets/IFExample.png" alt="Instant-Feed Example"/>
</div>

###Instructions###
[[Project Setup]]を実装した後に、下記の手順でAdColony Instant-Feed™広告を表示することができます。
####Step 1: Configure AdColony####
まず、AdColonyライブラリーを導入してください。
```java
import com.jirbo.adcolony.*;
```
次、以下のように設定メソッドを**main Activity** の onCreateメソッドの中で呼び出してください:
```java
AdColony.configure(this, client_options, app_id, zone_ids);
```
client_optionsをアプリの情報に入れ替えてください (e.g. "version:2.1,store:google" - 詳しくは [API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details) に参照してください) 。また、app_id/zone_idsを管理画面上から取得して下さい [Control Panel](http://clients.adcolony.com) ( "[Setting up Apps and Zones](http://support.adcolony.com/customer/portal/articles/761987-setting-up-apps-zones)" に参照). <br><br>
**Note:** この設定関数は**main Activity's** onCreateメソッドで **一回** のみ実行してください。 アプリケーションが実行する間にこのAcitivityの参照は廃棄されないようにしてください。

===
####Step 2: Pause and Resume####
アプリ中の全てのActivityにadcolonyのメソッドを呼び出すために、下記のようにonPause / onResumeでオーバーライドしてく下さい。
```java
public void onPause() 
{
  super.onPause();
  AdColony.pause(); 
}

public void onResume() 
{
  super.onResume();
  AdColony.resume( this ); 
}
```

===
####Step 3: AdColonyNativeAdViewオブジェクトの作成####
AdColonyNativeAdViewオブジェクトをリクエストしてアプリのレイアウトにそのプレースメントを作成する必要があります。パフォーマンスのためには、下記の動作はUI以外のスレードで実装してください(AsyncTasksが推奨)。**AdユニットをListViewに追加する場合、prepareForListView()メソッドを実行してくささい。**
```java
AdColonyNativeAdView ad = new AdColonyNativeAdView(activity, zone_id, width);
ad.prepareForListView(); //Call this if you will be adding into a ListView
if (ad.isReady())
{
  //Continue at Step 4
}
```
上記では、activityはアプリ内のActivityのリファレンスです、zone_idは管理画面から取ってconfigure段階で設定したzoneIDです、widthはadユニットが表示する必要な横幅です。広告が準備できたらプレースメントを組み立てます。

===

####Step 4: もっと情報を取得してプレースメントに入れる####
パブリックAPIを利用してもっと情報を取得してプレースメントに入れることができます。
```java
ad.getAdvertiserName(); //必須
ad.getAdvertiserImage(); //Optional
ad.getTitle(); //Optional
ad.getDescription(); //Optional
```

===
####Step 5: スポンサー標識の追加####
広告プレースメントにスポンサーという標識のテキストや画像を追加してください。demoアプリにシンプルな例が有りますので、ご参照してください。

===
####Step 6: Adユニットをアプリのレイアウトに追加####
Adユニットを作成できたら、アプリのレイアウトに追加してください、スクリーンに表示されたら再生できるはずです。注意したいところは、AdColonyNativeAdViewオブジェクトはハードウェアアクセラレーション有効になったらウェインドしか利用できないです。

ListViewに追加された場合、下記のように、アダプターのgetView()メソッドの中に、notifyAddedToListView()を実行してください。

```java
@Override
public View getView(int position, View view, ViewGroup parent)
{
  ...
  layout.addView(native_ad);
  native_ad.notifyAddedToListView();
  ...
  return your_view;
}
```

===
####Step 7: Adユニットを消す####
Adユニットを完全に完了した場合（例、新しいAdユニットをリプレースする場合、または完全にレイアウトから外す場合）、下記のようにdestroy()を実行してください。
```java
ad.destroy();
```
上記ではこのオブジェクトへの内部リファランスを消します、そしてGCでメモリーから開放することができます。<br><br>

**Note:** これは最小限で実装した例です, もっと詳しくは[API Details](https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details)またはサンプルアプリを参照してください。


##よくある質問##
###基本情報に関して###
- Q:各設定情報はどんな意味ですか
- A:
	- **App ID**: こちらは各アプリを指します。
	- **Zone ID**: こちらはアプリの下に紐づく、各掲載枠を指します。
	- **Call Back URL**: 動画再生の成果等を送るURLになります。設定していただかなくても本サービスはご利用頂けます。
	- **Custom ID**: CustomIDは、mediaのユーザーidを設定していただきます。設定した値はcall backを使用する場合、custom_idとして返します。

###SDK仕様に関して###
- Q: 縦画面の再生は可能か？
- A: AdColonyでは、スキップなし横画面フルサイズの再生となります。

- Q: AdColonyダイアログででポップアップの国別で表記を可能か？
- A: アプリの配信国に沿った各々の表記を行う場合、アプリ内でダイアログのご実装をご自身でして頂く必要がございます。

- Q: 上限回数は何回ですか？
- A: 上限回数は変更可能です。変更希望の場合、担当者もしくは(video-ad@glossom.co.jp)までご連絡下さい。

- Q: インセンティブはコイン以外でも可能か？
- A: 可能です。御社側で、動画視聴の成果通知に合わせてご実装して頂く必要がございます。

- Q: custom_idの設定する方法おしえてください。
- A: CustomIDの設定はSDK側でconfigure関数の前に、下記のように設定してください。

      AdColony.setCustomID("xxxx");
      https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details#setcustomid-string-id-


- Q:  V4VCのサーバー側のレスポンスと再送信
- A: 下記でございます。

		- "vc_success"
		正常に処理が終わり、ユーザーへのコイン付与が成功した場合、AdColony側から再送信を行いません。
	
		- "vc_decline" または "vc_noreward"
		正常に処理が終わったが、uidの誤り/不正と判断された場合、AdColony側から再送信を行いません。
	
		- [上記以外に、他にレスポンスされる場合]
		AdColonyは定期的に再送信行います。異常な場合以外は、こちら利用は控えて下さい。

###動画再生に関して###
- Q: 動画再生ができない場合どうすればいいですか
- A: 下記をチェックしてください。
	- 正しいIDは使われているか？
		- 発行されたApp ID/Zone IDの対象OSをお誤りのないよう、ご利用下さい。
	- IDと実装マニュアルの形式は同様か？
		- 動画リワードの場合[Showing V4VC Videos]
		- interstisialの場合[Showing Interstitial Videos]をご利用下さい。
	- 広告がダウンロードされていない
		- 再生を行うにあたって、動画をダウンロードする時間が必要です。
		- 広告準備が確認された後、再生を行って下さい。
		- 広告の再生可否をの検出方法を参考してください。

- Q: 広告の再生可否を検出するためには？
- A: 下記のdelegateメソッドは再生できる状態が変わった時点で通知が行われます。

         onAdColonyAdAvailabilityChange( boolean available, String zone_id )
         (https://github.com/AdColony/AdColony-Android-SDK/wiki/API-Details#onadcolonyadavailabilitychange-boolean-available-string-zone_id-)

- Q: 配信可能な広告がない（少ない）ですが、どうすればいいですか
- A: 広告在庫は様々なロジックで配信制限を行っております。詳細は下記を参考にして下さい。
	- ユーザーが不正と判断された場合
	- 1ユーザあたりの配信上限回数への到達
	- eCPMが極端に低い
	- ユーザー数が極端に少ない、リリース前もしくはリリース直後
	- androidの場合、GoogleのAdvertising IDを取得するため、プロジェクトの中にGoogle Play Services 4.0+ を追加してください。

###ストア申請に関して###
- Q: テスト切り替えの際の連絡はいつしたら良いか？
- A: リリース前に担当者へ配信切り替えのご連絡をして下さい。

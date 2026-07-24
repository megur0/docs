---
title: "TIPS - JavaScript"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(JavaScript)](./README.md) > TIPS


# 不具合トレース
* 例えば、`[vue-i18n] Value of key 'menu.loginPage' is not a string or function !` のような警告がコンソールに出ることがある。
* スタックトレースを追うと `AppHeader.vue?7568:465` のように表示されることがあるが、行数などはバンドル後のものになるため、実際のソースファイルと照らし合わせて確認する必要がある。
    * (IMO) このあたりはVue Devtoolsなどでソースマップ経由に自動的に対応してくれると助かる。


# event.preventDefault
* `event.preventDefault`メソッドは、イベントの発生元要素が持つデフォルトの動作をキャンセルするメソッド。
* 例えばフォームのsubmitイベントでは、デフォルトで`action`属性の指定先に画面遷移してしまうが、`preventDefault`を実行することでその画面遷移を止めることができる。
* (参考) https://qiita.com/yokoto/items/27c56ebc4b818167ef9e


# ローカルストレージ、Cookie、sessionStorage
* HTTPリクエストで自動的に送信されるのはCookieのみ。
* localStorageは基本的には消えないが、sessionStorageはタブを閉じると消える。
* (参考) https://qiita.com/terufumi1122/items/76bafb9eed7cfc77b798


# UUID → Base64の変換
* (参考) https://scrapbox.io/piyopiyo/JavaScript_%E3%81%A7_hex_string_%E3%81%A8_Base64_%E3%81%AE%E7%9B%B8%E4%BA%92%E5%A4%89%E6%8F%9B
* Base64はURLセーフではないため、URLとして扱う場合はさらに文字の変換が必要になる。
    * (参考) https://qiita.com/kunihiros/items/2722d690b1525813c45e

## (WIP) 自作を試みた際のメモ
* 2進数に変換して6bit文字列配列を作るところまで実装したが、上記のより良い方法を見つけたためそちらを採用した。
```js
// 例: UUID文字列（ハイフンを除いた32桁の16進数文字列）を2進数文字列に変換する
const uuid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
const binStr = uuid.split('-').join('').split('', 32)
    .map((hex) => parseInt(hex, 16))
    .map((hexNum) => hexNum.toString(2).padStart(4, '0'))
    .join('');
console.log(`binStr: ${binStr}`);

const bin6Arr = binStr.match(/.{6}/g);
console.log(`binStr6bit: ${bin6Arr}`);
```

## fromCharCodeについて
* (参考)
    * https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode
    * https://fdscaa.hatenablog.com/entry/2014/12/20/014748
    * https://stackoverflow.com/questions/9936490/can-i-pass-an-array-into-fromcharcode
    * https://stackoverflow.com/questions/8936984/uint8array-to-string-in-javascript
    * https://taiju.hatenablog.com/entry/20100515/1273903873
* 正規表現の参考: https://qiita.com/turmericN/items/0819317b5c075d971bfa

## その他
* キーワード: btoa, atob, base64エンコーダー自作
* (参考) https://news.mynavi.jp/techplus/article/zerojavascript-12/

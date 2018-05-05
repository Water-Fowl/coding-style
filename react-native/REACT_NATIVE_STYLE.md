# React Native コーディング規約

## Syntax
基本的には、eslintのreact-nativeに準拠する。
```
eslint . --fix
```
で自動fixできるので、ここでは割愛。

## 命名規則
クラス名、コンポーネント名はPascal Case
関数、変数など、その他のモジュールはCamel Caseで命名する。

### reduxのaction
reduxのactionの命名規則
- 値をstoreにいれるときは`set~`
- 値をstoreから外すときは`remove~`
- APIを叩くときは、、`[HTTP METHOD]~Request`と、`[HTTP METHOD]~Received`
  - 例1) ユーザーのデータを取ってくるときは、`getUserRequest`と`getUserReceived`
  - 例2) 試合のデータをdbに保存したいときは、`postGameRequest`と`postGameReceived`

## import規則
基本的には、import行が肥大化するのが好ましくないため、一行ですませる。
```
import { mapStateToProps, errorAlertCallback } from "utils";
```
ただ、一行では収まらない場合は、
```
import {
  AsyncStorage, Dimensions, Image, StyleSheet,
  Text, TextInput, TouchableOpacity, View
} from "react-native";
```
こんな感じで分割して書く。

`src/modules/`内の関数をimportするときは、
```
import * as authenticationModules from "../../modules/authentication";
```
という感じでimportする。

# Atomic Designのガイドライン

## atoms

### 概要
react-nativeのコンポーネントのみで作成され、再利用される汎用的なコンポーネント。

### 原則
- stateはもたない
- storeも参照しない
- atomsを中で使わない

## molecules

### 概要
atomsを使って構成されるコンポーネント。
意味を持ったコンポーネントが構成される。

### 原則
- stateはもたない
- storeも参照しない
- atomsを中で使う。使わない場合はatomsへ。

## organisms
画面に依存したコンポーネント。
再利用性のためではなく、可読性をあげるために切り出す。

### 原則
以下のどれかに当てはまった場合は、moleculesではなくorganismsになる。
- stateを持つ
- reduxのstoreを参照している
- reduxのactionを実行している
- moleculesを使って書かれている

## templates
画面全体を表現したコンポーネント。

### 原則
- 基本的には、他のatomicなコンポーネントと、レイアウトを記述する`View`で構成される
- render関数の中身は極力少なくして、可読性をあげる
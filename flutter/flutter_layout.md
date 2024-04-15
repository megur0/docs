- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 公式ドキュメント
* https://docs.flutter.dev/development/ui/layout
* https://docs.flutter.dev/development/ui/layout/constraints

# 原則: 制約は親から子へ。子が希望のサイズを伝える。親がポジションを決定。
> Constraints go down. Sizes go up. Parent sets position.
* あるウィジェットについて、そのウィジェットのレイアウトの制約は親が決める。
    * （制約：a minimum and maximum width, and a minimum and maximum height.）
* そのレイアウトに基づいて、あるウィジェットはその範囲内で自分のサイズを決定し、親に伝える。
* そして親によって自分のpositionが決定される。
    * （position: horizontally in the x axis, and vertically in the y axis) 
    * なお、自分のウィジェットのサイズを決める前に、自分の子ウィジェットに対して同様にレイアウトを伝えてサイズを受け取りpositionを決定する。

# 制限: ウィジェット自身はポジションを決められない、知り得ない。
* ウィジェット自身は自分のスクリーン上のpositionは決められないし知り得ない。
* あるウィジェットのサイズやpositionはそれ単体のみで決定ことは不可能で、ウィジェットツリー全体を以て決定する。
* したがって、子がいくらサイズを指定しても、ignoreされることがある。

# 個別のウィジェットの仕様を確認する必要がある。
* 上記の原則を理解していたとしても、最終的な自身のサイズの決定や子ウィジェットのpositionの決定はそのウィジェットの仕様に委ねられるため、それについては個々のドキュメントを読む。
    * 仕様はflutterのコードを読むことでも理解可能。
    * 具体的には対象のウィジェットのRenderObjectのperformLayout()関数の処理




# Online Chat Messenger
クライアント-サーバー間のTCP通信によってチャットルームを作成し、UDP通信によって同じ部屋に参加しているユーザー同士でメッセージを送信できるプログラムです。CLIから起動できます。


# 要件整理
# 11.25 Sat - 30 Thur - 各自でステージ１に取り掛かる
<br />

ステージ１：クライアント->サーバーにUDP通信で送信したメッセージが全クライアントに共有される
<br/>



## 機能要件
タスクが完了したらcheckを入れてください。

- [x]  local chat messengerとほぼ同じです。 

- [x] セッションが開始される際には、クライアントはユーザーにユーザー名を入力させます。  

- [x] メッセージの送信プロトコルは比較的シンプルです。メッセージの最初の1バイト、usernamelenは、ユーザー名の全バイトサイズを示し、これは最大で255バイト（2^8 - 1バイト）になります。サーバはこの初めのusernamelenバイトを読み、送信者のユーザー名を特定します。その後のバイトは送信される実際のメッセージです。この情報はサーバとクライアントによって自由に使用され、ユーザー名の表示や保存が可能です。

- [x] バイトデータはUTF-8でエンコードおよびデコードされます。これは、1文字が1から4バイトで表現される意味です。Pythonのstr.encodeとstr.decodeメソッドは、デフォルトでこの挙動を持っています。 

- [x] サーバにはリレーシステムが組み込まれており、現在接続中のすべてのクライアントの情報を一時的にメモリ上に保存します。新しいメッセージがサーバに届くと、そのメッセージは現在接続中の全クライアントにリレーされます。

- [x] クライアントは、何回か連続で失敗するか、しばらくメッセージを送信していない場合、自動的にリレーシステムから削除されます。この点でTCPと異なり、UDPはコネクションレスであるため、各クライアントの最後のメッセージ送信時刻を追跡する必要があります。  


# 11.30 thurs - - stage2に取り掛かる


ステージ2 : chat roomの作成/接続

<br/>

## 機能要件
タスクが完了したらcheckを入れてください。


1. チャットルームの作成と接続（TCP）
   
- [x] 担当　kanata99
- [x] カスタムTCPプロトコルを作成し、クライアントとサーバがチャットルームの作成と接続のために通信できるようにします。TCPは、メッセージの受信と順序が保証されるため、信頼性が高いです。以下は、チャットルームプロトコル（TCRP）と呼ぶカスタムプロトコルです。

     - [x] ヘッダー（32バイト）：RoomNameSize（1バイト） | Operation（1バイト） | State（1バイト） | OperationPayloadSize（29バイト）

     - [x] ボディ：最初のRoomNameSizeバイトがルーム名で、その後にOperationPayloadSizeバイトが続きます。ルーム名の最大バイト数は2^8バイトであり、OperationPayloadSizeの最大バイト数は2^29バイトです。

- [x] 担当　yadonさん
- [x] RoomNamesはUTF-8でエンコード/デコードされます。OperationPayloadは、操作と状態に応じて異なる方法でデコードされる可能性があります（整数、文字列、JSONファイルなど）   

- [x] 担当　yukimiさん
- [x] 新しいチャットルームを作成する場合、操作コードは1です。0はリクエスト、1は準拠、2は完了です。TCPは完全なトランザクションを保証するために使用されます。

     - [x] サーバの初期化（0）：クライアントが新しいチャットルームを作成するリクエストを送信します。ペイロードには希望するユーザー名が含まれます。

     - [x] リクエストの応答（1）：サーバはステータスコードを含むペイロードで即座に応答します。

     - [x] リクエストの完了（2）：サーバは特定の生成されたユニークなトークンをクライアントに送り、このトークンにユーザー名を割り当てます。このトークンはクライアントをチャットルームのホストとして識別します。トークンは最大255バイトです。

- [x] 担当　yukimiさん
- [x] 新しいチャットルームに参加しようとするとき、操作コードは2です。状態は作成時と同様で、クライアントも生成されたトークンを受け取りますが、ホストではありません。

- [x] チャットルームごとに、サーバは許可されたリストトークンのリストを追跡する必要があります。ユーザーがそのトークンで参加すると、トークンの所有者として設定されます。メッセージがチャットルーム内の他のすべての人にリレーされるためには、トークンとIPアドレスが一致しなければなりません。

- [x] チャットルームの作成またはチャットルームへの参加が完了すると、TCPコネクションは終了します。クライアントは自動的にUDPでサーバに接続します。

2. ルームでのチャット（UDP）
   
- [x] yadonさん
- [ ] チャットルームはホストが存在する限り有効です。ホストが退出すると、チャットルームは自動的に閉じられます。
- [x] チャットルームに参加するには、専用の許可トークンが必要です。このトークンは参加者のIPアドレスと一致する必要があります。
   
- [x] kanata
- [x] クライアントとサーバ間でのメッセージ交換は、すべてUDPプロトコルを使用して行われます。
   
- [x] yadonさん
- [x] クライアントがサーバに送信するパケットは、最大4096バイトのメッセージとなります。そのうちの最初の2バイトは、ルーム名とトークンのバイトサイズを示しています。
- [x] ヘッダー：RoomNameSize（1バイト）| TokenSize（1バイト）
- [x] ボディ：最初のRoomNameSizeバイトはルーム名、次のTokenSizeバイトはトークン文字列、そしてその残りが実際のメッセージです。
   
- [x] yukimiさん
- [x] クライアントはサーバから最大で4094バイトのパケットを受信できます。これはメッセージのみで、ヘッダーは含まれません。
クライアントがリレーシステムから削除された場合、そのトークンもサーバから削除されます。サーバは切断メッセージを該当のクライアントに転送します。その後、クライアントは再度チャットルームに参加する必要があります。




# 議事録
# 12/1 stage2進捗報告
## 参加者
- yadon
- yukimi
## 進捗状況
- yadon: 進捗なし(詰まっているというより時間確保できていない)    
- yukimi: 担当分の大枠の検討を進めた    
## 疑問点等
開発の進め方について：各々で分担したソースコードをマージしていく具体的な流れを確認したい
→ 次回進捗報告にて確認
その他詰まったところがあれば適宜 Discord で相談

# 12/3 stage2進捗報告
## 参加者
- yadon
- yukimi
- kanata99


## 進捗状況
kanata99: 大枠担当箇所は完成した  
yukimi: 担当箇所終了 pushでのエラー解決をしていく  
yadon: 大枠終了  

## 疑問点等
### 開発の進め方について
主な流れについては

1. Issueを作って自分のタスクを登録する
2. developブランチにて```git pull```を実行
3. ```git switch -c ブランチ名```で追加機能用のブランチを作成、移動
4. 追加機能作成
5. ```git add .```で修正したファイルをgitに指示する（ステージング）
6. ```git commit -m "変更内容に関するメッセージ"```でステージングしたファイルの内容をgitに指示する
7. ```git push origin ブランチ名```でリモートリポジトリへ一連の作業内容を保存
8. developブランチへプルリクエストを作成する
9. プルリクエストに対するレビューをしてもらう
10. developブランチへマージする

# 12/4 stage2進捗報告
## 参加者
- yadon
- yukimi
- kanata99

## 進捗状況
kanata99: server.pyの操作で修正アリ 
yukimi: 担当箇所終了  
yadon: 担当箇所終了

## 今後の方針
問題なくstage2は終われそうなのでstage3のパスワードの設定とデスクトップアプリの作成終了まで行けると　　
いいアプリケーションが作れるのでここを終了目標とした。

## 疑問点等
特になし

# 12/6 stage2進捗報告
## 参加者
- yadon
- yukimi
- kanata99

## 進捗状況
kanata99: serverのクラスをTCPとUDPで分ける/bodyが最大バイトを超えたときの処理
yukimi:  2/3終了
yadon: 1/2終了

## 疑問点等
IPアドレスをどう一致させるか？  
→リストを作ってIPアドレスを受け渡し可能にする
クライアントとサーバ-の処理に関して
→chatroomはUDPで行われるのでUDPサーバ-を作る

# 12/8 stage2進捗報告
## 参加者
- yadon
- yukimi
- kanata99

## 進捗状況
kanata99: 終了.yukimiさんとyadonさんのコードをmergeしたあとclient.pyをtcp/udpのクラスに分ける作業に入る
yukimi:  終了
yadon: 大枠終了

## 疑問点等
特になし


# 12/10 stage2進捗報告
## 参加者
- yadon
- yukimi
- kanata99

## 進捗状況
kanata99: tcp/udpのclientに分ける処理　終了
yukimi:  2終了
yadon: 終了

stage2の要件は8割程度終了したが、細かい論理エラーが多くその部分の修正を必要である。
## 疑問点等
tcp通信上のデータ受け渡しが空になってしまう
udp通信上のデータ受け渡しが空になってしまう

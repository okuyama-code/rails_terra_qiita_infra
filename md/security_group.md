
```tf
# アプリケーション用セキュリティグループ
resource "aws_security_group" "prod_sg_app" {
  name        = "${var.r_prefix}-sg-app"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.prod_vpc.id

  # ALBからのトラフィックのみを許可
  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.prod_sg_alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.r_prefix}-sg-app"
    Environment = "Production"
  }
}

# ロードバランサー用セキュリティグループ
resource "aws_security_group" "prod_sg_alb" {
  name        = "${var.r_prefix}-sg-alb"
  description = "Security group for application load balancer"
  vpc_id      = aws_vpc.prod_vpc.id

  # HTTPSのみを許可
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 必要に応じて制限可能
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.r_prefix}-sg-alb"
    Environment = "Production"
  }
}

# データベース用セキュリティグループ
resource "aws_security_group" "prod_sg_db" {
  name        = "${var.r_prefix}-sg-db"
  description = "Security group for database"
  vpc_id      = aws_vpc.prod_vpc.id

  # アプリケーションサーバーからのアクセスのみを許可
  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.prod_sg_app.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.r_prefix}-sg-db"
    Environment = "Production"
  }
}
```

この改善されたセキュリティグループ設定について、詳細に説明します：

アプリケーション用セキュリティグループ（prod_sg_app）:

インバウンド：ALBからの80番ポートへのトラフィックのみを許可しています。これにより、インターネットからの直接アクセスを防ぎます。
アウトバウンド：全ての出力トラフィックを許可しています（必要に応じて制限可能）。


ロードバランサー用セキュリティグループ（prod_sg_alb）:

インバウンド：HTTPSの443番ポートのみを開放しています。本番環境では暗号化されたトラフィックのみを受け付けるべきです。
アウトバウンド：全ての出力トラフィックを許可しています。


データベース用セキュリティグループ（prod_sg_db）:

インバウンド：アプリケーションサーバーからの3306番ポート（MySQL）へのアクセスのみを許可しています。
アウトバウンド：全ての出力トラフィックを許可しています（必要に応じて制限可能）。



主な改善点：

最小権限の原則：必要最小限のポートとプロトコルのみを開放しています。
セキュリティグループの連鎖：セキュリティグループ同士を参照することで、より細かい制御を実現しています。
HTTPS の使用：ALBでHTTPSを使用し、暗号化されたトラフィックのみを受け付けています。
データベースの保護：データベースへのアクセスをアプリケーションサーバーからのみに制限しています。

追加の推奨事項：

VPC設計：プライベートサブネットとパブリックサブネットを適切に使い分けてください。
ネットワークACL：サブネットレベルでの追加のセキュリティ層として使用を検討してください。
WAF（Web Application Firewall）：ALBの前に配置し、一般的なWeb攻撃から保護することを検討してください。
ログ監視：CloudTrailやVPCフローログを有効にし、定期的に監視してください。
暗号化：転送中および保存中のデータの暗号化を実装してください。
定期的なセキュリティ監査：設定を定期的にレビューし、必要に応じて更新してください。

注意点：

この設定はベースラインであり、具体的なアプリケーションの要件に応じてさらなる調整が必要な場合があります。
セキュリティは継続的なプロセスです。新しい脅威や変化するアプリケーションの要件に応じて、定期的に設定を見直し、更新することが重要です。

このような設定により、本番環境のセキュリティを大幅に向上させることができます。ただし、セキュリティは多層的なアプローチが必要であり、セキュリティグループはその一部に過ぎないことを忘れないでください。


## TCPとは
https://www.itmanage.co.jp/column/tcp-ip-protocol/
https://wa3.i-3-i.info/word19.html

TCPの主な特徴：

コネクション型：

通信を開始する前に、送信者と受信者の間で接続（コネクション）を確立します。
この過程を「3ウェイハンドシェイク」と呼びます。


信頼性：

データの到着を保証します。
受信側は、受け取ったデータの確認応答（ACK）を送信します。


順序制御：

データパケットに番号を付けて送信し、受信側で正しい順序に並べ替えます。


フロー制御：

受信側の処理能力に合わせて、送信速度を調整します。


エラー検出と再送：

データの破損や喪失を検出し、必要に応じて再送します。


全二重通信：

双方向の同時通信が可能です。



TCPの動作手順：

コネクション確立（3ウェイハンドシェイク）：

クライアントがSYN（同期）パケットを送信
サーバーがSYN-ACKで応答
クライアントがACK（確認応答）を送信


データ転送：

データをパケットに分割して送信
受信側は各パケットにACKで応答


コネクション終了：

FIN（終了）パケットを交換して接続を閉じる



TCPの用途：

Webブラウジング（HTTP/HTTPS）
電子メール（SMTP、POP3、IMAP）
ファイル転送（FTP）
リモート接続（SSH）

TCPとUDPの比較：

TCPは信頼性と順序を重視し、UDPは速度を重視します。
TCPはストリーミングデータに適し、UDPはリアルタイム通信（ビデオ通話など）に適しています。

セキュリティの観点：

TCPは接続型のため、接続の開始と終了を明確に把握できます。
ファイアウォールやセキュリティグループでは、特定のTCPポートの開閉を制御することで、アプリケーションレベルでのアクセス制御が可能です。

TCPの理解は、ネットワークセキュリティや効率的なアプリケーション設計において非常に重要です。セキュリティグループの設定や、アプリケーションの通信プロトコル選択の際に、TCPの特性を考慮することで、より安全で効率的なシステム設計が可能になります。


```
sequenceDiagram
    participant クライアント
    participant サーバー

    Note over クライアント,サーバー: 1. 3ウェイハンドシェイク
    クライアント->>サーバー: SYN
    サーバー->>クライアント: SYN-ACK
    クライアント->>サーバー: ACK

    Note over クライアント,サーバー: 2. データ転送
    クライアント->>サーバー: データ パケット1
    サーバー->>クライアント: ACK 1
    クライアント->>サーバー: データ パケット2
    サーバー->>クライアント: ACK 2

    Note over クライアント,サーバー: 3. コネクション終了
    クライアント->>サーバー: FIN
    サーバー->>クライアント: ACK
    サーバー->>クライアント: FIN
    クライアント->>サーバー: ACK
```
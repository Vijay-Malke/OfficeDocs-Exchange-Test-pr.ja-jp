﻿---
title: OWA 正常性セットのトラブルシューティング
TOCTitle: OWA 正常性セットのトラブルシューティング
ms:assetid: eae9dbd7-2ce6-43ce-b9a1-b114fd0c3ab4
ms:mtpsurl: https://technet.microsoft.com/ja-jp/library/ms.exch.scom.owa(v=EXCHG.150)
ms:contentKeyID: 53181844
ms.date: 01/28/2016
mtps_version: v=EXCHG.150
ms.translationtype: HT
---

# OWA 正常性セットのトラブルシューティング

 

_**適用先:** Exchange Server 2013_

_**トピックの最終更新日:** 2015-03-09_

Outlook Web App (OWA) 正常性セットは、Outlook Web App サービスの全体的な状態を監視します。

Outlook Web App が正常でないことを示す警告を受け取った場合、ユーザーが Outlook Web App を使用して自分のメールボックスにアクセスできなくなる可能性がある問題を示しています。

## 説明

Outlook Web App サービスは、次のプローブとモニターを使用して監視されます。


<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Probe</th>
<th>正常性セット</th>
<th>依存関係</th>
<th>関連するモニター</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>OwaCtpProbe</p></td>
<td><p>Outlook Web App</p></td>
<td><p>Active Directory</p>
<p>インフォメーション ストア</p></td>
<td><p>OwaCtpMonitor</p></td>
</tr>
</tbody>
</table>


プローブとモニターの詳細については、「[サーバーの状態とパフォーマンス](https://technet.microsoft.com/ja-jp/library/jj150551\(v=exchg.150\))」を参照してください。

## 一般的な問題

このプローブは、いくつかの理由で失敗することがあります。以下は、よくある理由の一部です。

  - 監視対象のクライアント アクセス サーバー (CAS) でホストされている Outlook Web App アプリケーション プールが応答していないか、メールボックス サーバーでホストされているアプリケーション プールが応答していない。

  - CAS でネットワークの問題が発生しており、CAS がメールボックス サーバーやドメイン コントローラーに接続できない。

  - 監視アカウントの資格情報が正しくない。

  - ユーザーのデータベースがマウントされていないか、そのメールボックスのインフォメーション ストアにアクセスできない。

  - インフォメーション ストアが応答していない。

  - ドメイン コントローラーが応答していない。

## ユーザー操作

サービスは、警告の発行後に回復することがあります。そのため、正常性セットが異常であることを示す警告を受け取ったときは、まず、その問題がまだ存在しているかどうかを確認します。問題が存在する場合は、次のセクションで説明する適切な回復操作を実行します。

## 問題がまだ存在していることを確認する

1.  警告に記載された正常性セット名とサーバー名を確認します。

2.  メッセージの詳細には、警告の正確な原因に関する情報が示されています。ほとんどの場合、根本原因を特定するためのトラブルシューティング情報としては、メッセージの詳細だけで十分です。メッセージの詳細が不明確な場合は、次の操作を行います。
    
    1.  Exchange 管理シェル を開き、次のコマンドを実行して、警告を生成した正常性セットの詳細を取得します。
        
            Get-ServerHealth <server name> | ?{$_.HealthSetName -eq "<health set name>"}
        
        server1.contoso.com に関する Outlook Web App 正常性セットの詳細を取得するには、次のコマンドを実行します。
        
            Get-ServerHealth server1.contoso.com | ?{$_.HealthSetName -eq "OWA"}
    
    2.  コマンド出力を確認して、エラーを報告したモニターを特定します。警告を発行したモニターの **AlertValue** の値は `Unhealthy` です。
    
    3.  正常状態にないモニターに関連するプローブを再実行します。関連するプローブについては、「説明」セクションの表を参照してください。このためには、次のコマンドを実行します。
        
            Invoke-MonitoringProbe <health set name>\<probe name> -Server <server name> | Format-List
        
        たとえば、*server1.contoso.com* の Exchange ActiveSync 監視プローブを作成するには、次のコマンドを実行します。
        
            Invoke-MonitoringProbe -Identity ActiveSync.Protocol\ActiveSyncSelfTestProbe -Server server1.contoso.com
    
    4.  コマンド出力で、プローブの **Result** の値を確認します。値が **Succeeded** であれば、この問題は一時的なエラーであり、もう存在しません。値がそれ以外の場合は、次のセクションで説明する回復手順を参照してください。

## OwaCtpMonitor の回復操作

正常性セットからの電子メール警告には、次の情報が含まれます。

  - 警告を送信したサーバーの名前

  - 最後のエラーの完全な例外の追跡 (診断データおよび特定の HTTP ヘッダー情報を含む)  
    
    **注**   完全な例外の追跡に記載された情報を使用して、問題をトラブルシューティングできます。プローブによって生成された例外には、プローブが失敗した理由を説明する「エラーの理由」が含まれています。たとえば、例外には以下が含まれます。
    
      - **MissingKeyword**   予期したキーワードがサーバーの応答内にありませんでした。この場合、例外に予期したキーワードが含まれています。
    
      - **NameResolution**   指定されたサーバー名を DNS で解決できません。
    
      - **NetworkConnection**   プローブが CAFE 上の OWA アプリケーション プールに接続しようとしたときに、ネットワーク接続エラーを受け取っています。
    
      - **UnexpectedHttpResponseCode**   予期しない HTTP コードが応答に含まれていました。たとえば、サーバーが **503** HTTP コードを返しました。
    
      - **RequestTimeout**   サーバーの応答が、クライアント要求に対して時間がかかり過ぎています。
    
      - **ScenarioTimeout**   プローブは正常に終了しましたが、その処理にかかった時間が 1 分を超えていました。これは、通常、システムが過負荷になっていることを示しています。
    
      - **OwaErrorPage**Outlook Web App がエラー ページを返しました。障害の原因となったエラーの名前は、通常、例外メッセージに記載されています。
    
      - **OwaMailboxErrorPage**Outlook Web App が、メールボックス ストア関連のエラーを含むエラー ページを返しました。これは、通常、メールボックス ストアがダウンしているか、メールボックスがマウント解除中であることを示します。
    
    例外の追跡には、**FailingComponent** という重要なフィールドが含まれています。プローブは、次の例のように、エラーの特定を試みます。
    
      - **Mailbox**   プローブは Outlook Web App に到達できますが、メールボックス ストアに接続できません。この場合、プローブが失敗したか、またはメールボックス アクセスの遅延が原因でプローブが失敗し **ScenarioTimeout** エラーが発生しています。この種のエラーが発生する場合は、メールボックス サーバーの状態をチェックする必要があります。
    
      - **Active Directory**   プローブは Outlook Web App に到達できますが、Active Directory に接続できません。この場合、プローブが失敗したか、または Active Directory 呼び出しの遅延が原因でプローブがタイムアウトした可能性があります。この種のエラーが発生する場合は、ドメイン コントローラーの状態をチェックすると共に、CA サーバーおよびメールボックス サーバーとドメイン コントローラーとの間のネットワーク接続をチェックする必要があります。
    
      - **Owa**   これは、一般に、Outlook Web App 層内でエラーが発生したことを意味します。この種のエラーが発生する場合は、CA サーバーおよびメールボックス サーバー上の Outlook Web App プロセスの状態を確認すると共に、ネットワーク接続をチェックする必要があります。
    
    例外には、最後の HTTP 要求と、プローブが失敗する前に受信した応答情報も含まれています。エスカレーションの本文には、プローブのログへのパスが含まれています。その情報を使用して、完全な HTTP Web 要求と、プローブが失敗したときに送信された応答を確認できます。失敗した操作だけが記録されるため、このファイルに含まれるのは失敗したプローブのデータだけです。この情報を利用すると、テストの失敗原因をより完全に把握できます。

  - 可用性メトリックがどの程度下がったか (x %)。

  - プローブの HTTP 要求の完全な追跡を含むフォルダーへの完全なパス。この情報は、既定では *\<Exchange サーバー\>*\\Logging\\Monitoring\\OWA\\ClientAccessProbe フォルダーにあります。

  - 警告の発生日時。

この問題をトラブルシューティングするには、次の手順に従います。

1.  テスト ユーザー アカウントを作成し、そのテスト ユーザー アカウントを使って CAS にログオンします。たとえば、https:// *\<サーバー名\>*/owa を使用してログオンします。
    
    これが失敗する場合は、別の CA サーバーを使用してテストを行い、メールボックス サーバーではなく特定の CAS で問題が発生していることを確認します。

2.  CA およびメールボックス サーバー間のネットワーク接続を確認します。ping.exe を使用して、各サーバーが応答していることを確認します。

3.  OWA.Protocol 正常性セットの警告に、特定のメールボックス サーバーに影響する問題を示しているものがないかどうかを確認します。詳細については、「[OWA.Protocol 正常性セットのトラブルシューティング](troubleshooting-owa-protocol-health-set.md)」を参照してください。

4.  IIS マネージャーを起動し、問題を報告しているサーバーに接続して、MSExchangeOwaAppPool アプリケーション プールが CAS 上で実行されていることを確認します。

5.  IIS マネージャーで、既定の Web サイトが実行されていることを確認します。

6.  失敗したプローブのメールボックス データベースを特定し、そのメールボックス データベースがメールボックス サーバーでアクティブであることと、メールボックス ストアが正常であることを確認します。失敗したデータベースの GUID 情報を見つけるには、完全な例外の追跡情報を開きます。各エラーには、次の例のようなエントリが含まれます。
    
    Owa プローブを開始しています。対象:https://localhost/owa/、ユーザー名:*HealthMailboxdf8b87828ab0427cb91e985bbdfcec62@yourdomain.com*

7.  HealthMailbox GUID をコピーしてから、シェルで次のコマンドを実行します。
    
        Get-Mailbox -Monitoring -Identity <username>
    
    たとえば、次のコマンドを実行します。
    
        Get-Mailbox -Monitoring -Identity HealthMailboxdf8b87828ab0427cb91e985bbdfcec62@yourdomain.com
    
    返されたオブジェクトで、ユーザーのデータベース名を特定できます。また、現在アクティブなデータベースがある場所も確認できます。

8.  サイト間のリダイレクトを構成している場合は、プローブが失敗し、MissingKeyword エラーが発生する可能性があります。この現象が発生するのは、既定では、CA プローブが任意の場所のアカウントで実行されることが原因です。また、リダイレクトを使用するときにプローブが別のサイト上の CAS をテストしないこともその原因です。この問題を解決するには、各サイト上のサーバーを MonitoringGroups に含めます。ある監視グループ内の CA サーバーは、常に同じグループ内のメールボックス サーバーと一緒にテストされます。
    
    サーバーの監視グループを特定するには、次のコマンドを実行します。
    
        Get-ExchangeServer | ft MonitoringGroup
    
    サーバーの監視グループを変更するには、*MonitoringGroup* パラメーターを指定して **Set-ExchangeServer** コマンドレットを使用します。たとえば、次のように使用します。
    
        Set-ExchangeServer -Identity "ServerName" -MonitoringGroup "Primary"

9.  IIS マネージャーで <strong>アプリケーション プール</strong> をクリックし、Exchange 管理シェル から次のコマンドを実行して、**MSExchangeOWAAppPool** アプリケーション プールをリサイクルします。
    
        %SystemRoot%\System32\inetsrv\Appcmd recycle MSExchangeOWAAppPool

10. 関連するプローブを再実行します (「問題がまだ存在していることを確認する」セクションの手順 2c を参照)。

11. 問題がまだ存在している場合は、IISReset ユーティリティを使用するか、次のコマンドを実行して、IIS サービスをリサイクルします。
    
        Iisreset /noforce

12. 関連するプローブを再実行します (「問題がまだ存在していることを確認する」セクションの手順 2c を参照)。

13. 問題がまだ存在している場合は、サーバーを再起動します。

14. サーバーが再起動したら、関連するプローブを再実行します (「問題がまだ存在していることを確認する」セクションの手順 2c を参照)。

15. プローブがまだ失敗する場合、この問題の解決にサポートが必要なこともあります。この問題を解決するには、Microsoft のサポート担当者にお問い合わせください。Microsoft のサポート担当者に問い合わせるには、「[Exchange Server サポート ページ](http://go.microsoft.com/fwlink/p/?linkid=180809)」にアクセスしてください。ナビゲーション ウィンドウで、<strong>サポート オプションとリソース</strong> をクリックし、<strong>テクニカル サポートを利用する</strong> に表示されるいずれかのオプションを使用して、Microsoft のサポート担当者に問い合わせます。組織には Microsoft 製品サポート サービスに直接問い合わせるための特定の手順がある場合があるので、組織のガイドラインを最初に必ず確認してください。

## 詳細情報

[Exchange 2013 の新機能](https://technet.microsoft.com/ja-jp/library/jj150540\(v=exchg.150\))

[Exchange 2013 コマンドレット](https://technet.microsoft.com/ja-jp/library/bb124413\(v=exchg.150\))


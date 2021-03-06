﻿---
title: 'ハイブリッド展開用に Exchange 2013 パブリック フォルダーを構成する: Exchange Online Help'
TOCTitle: ハイブリッド展開用に Exchange 2013 パブリック フォルダーを構成する
ms:assetid: b828520f-022c-4fcb-ab68-e1c330e87c33
ms:mtpsurl: https://technet.microsoft.com/ja-jp/library/Dn986544(v=EXCHG.150)
ms:contentKeyID: 65452438
ms.date: 05/22/2018
mtps_version: v=EXCHG.150
ms.translationtype: HT
---

# ハイブリッド展開用に Exchange 2013 パブリック フォルダーを構成する

 

_<strong>適用先:</strong>Exchange Server 2013, Exchange Server 2016_

**概要**:Exchange 2013 環境にあるオンプレミスのパブリック フォルダーに Exchange Online ユーザーがアクセスできるようにする手順。

ハイブリッド展開では、ユーザーは Exchange Online と Exchange オンプレミスのいずれか一方または両方に配置され、パブリック フォルダーは Exchange Online か Exchange オンプレミスのいずれかに配置されます。オンライン ユーザーが、Exchange Server 2013 のオンプレミス環境のパブリック フォルダーにアクセスすることが必要な場合があります。同様に、Exchange 2013 ユーザーが、Office 365 または Exchange Online のパブリック フォルダーにアクセスすることが必要な場合もあります。


> [!NOTE]
> Exchange 2010 または Exchange 2007 のパブリック フォルダーがある場合は、「<A href="https://docs.microsoft.com/ja-jp/exchange/collaboration-exo/public-folders/set-up-legacy-hybrid-public-folders">ハイブリッド展開用に従来の社内パブリック フォルダーを構成する</A>」をご覧ください。



この記事では、Exchange Online/Office 365 のユーザーが Exchange 2013 のパブリック フォルダーにアクセスできるようにする方法について説明します。オンプレミス Exchange 2013 のユーザーが Exchange Online のパブリック フォルダーにアクセスできるようにする方法については、「[ハイブリッド展開用に Exchange Online パブリック フォルダーを構成する](https://docs.microsoft.com/ja-jp/exchange/collaboration-exo/public-folders/set-up-exo-hybrid-public-folders)」をご覧ください。

Exchange Online/Office 365 のユーザーが Exchange 2013 のパブリック フォルダーにアクセスするには、そのユーザーがオンプレミスの Exchange 環境で MailUser オブジェクトによって表されている必要があります。この MailUser オブジェクトは、ターゲットの Exchange 2013 パブリック フォルダー階層に対してローカルである必要もあります。現在、オンプレミスで MailUser オブジェクトとして表されていない Office 365 ユーザーがある場合は、Microsoft サポート技術情報 3106618「[Exchange Online ユーザーがレガシ パブリック フォルダーにアクセスできない](https://go.microsoft.com/fwlink/p/?linkid=699451)」を参照し、対応するオンプレミスのエンティティを作成してください。

## 始める前に把握しておくべき情報

1.  以下の手順では、ハイブリッド構成ウィザードを使って、社内環境と Exchange Online 環境の構成と同期を行うこと、およびほとんどのユーザーの自動検出に使われる DNS レコードが社内のエンドポイントを参照していることを前提としています。詳細については、「[ハイブリッド構成ウィザード](hybrid-configuration-wizard-exchange-2013-help.md)」を参照してください。

2.  これらの手順は、オンプレミスの Exchange サーバー上で Outlook Anywhere が有効になっており、機能していることを前提としています。Outlook Anywhere を有効にする方法については、「[Outlook Anywhere](https://technet.microsoft.com/ja-jp/library/bb123741\(v=exchg.150\))」をご覧ください。

3.  Exchange と Office 365 のハイブリッド展開におけるパブリック フォルダーの共存の実装では、インポート手順中の競合を修正することが必要になる場合があります。競合は、メールを有効にしたパブリック フォルダーにルーティング不可能な電子メール アドレスが割り当てられていることや、Office 365 の他のユーザーまたはグループとの競合、あるいはその他の原因により発生する可能性があります。

4.  複数の場所にまたがってパブリック フォルダーにアクセスするには、Outlook クライアントを 2012 年 11 月以降の Oooklook パブリック更新プログラムにアップグレードする必要があります。
    
    1.  Outlook 2010 用の 2012 年 11 月の Oooklook 更新プログラムをダウンロードするには、「[Microsoft Outlook 2010 (KB2687623) 32 ビット版の更新プログラム](https://www.microsoft.com/ja-jp/download/details.aspx?id=35702)」を参照してください。
    
    2.  Outlook 2007 用の 2012 年 11 月の Outlook 更新プログラムをダウンロードするには、「[Microsoft Office Outlook 2007 (KB2687404) の更新プログラム](https://www.microsoft.com/ja-jp/download/details.aspx?id=35718)」を参照してください。

5.  Outlook 2011 for Mac と Outlook for Mac for Office 365 ではクロスプレミスのパブリック フォルダーがサポートされていません。Outlook 2011 for Mac や Outlook for Mac for Office 365 を利用してパブリック フォルダーにアクセスする場合、ユーザーはパブリック フォルダーと同じ場所にいる必要があります。また、メールボックスが Exchange Online にあるユーザーは、Outlook Web App を使ってオンプレミスのパブリック フォルダーにアクセスできなくなります。
    

    > [!NOTE]
    > Outlook 2016 for Mac ではクロスプレミスのパブリック フォルダーがサポートされています。組織内のクライアントが Outlook 2016 for Mac を使用している場合は、2016 年 4 月の更新プログラムをインストールしていることを確認してください。そうしないと、それらのユーザーはハイブリッド トポロジ内のパブリック フォルダーにアクセスすることができません。詳細については、「<A href="https://technet.microsoft.com/ja-jp/library/mt788631(v=exchg.150)">Outlook 2016 for Mac によるパブリック フォルダーへのアクセス</A>」を参照してください。



## 手順 1: スクリプトをダウンロードする

1.  次のファイルを、「[Mail-enabled Public Folders - directory sync script](https://www.microsoft.com/en-us/download/details.aspx?id=46381)」からダウンロードします。
    
      - `Sync-MailPublicFolders.ps1`
    
      - `SyncMailPublicFolders.strings.psd1`

2.  PowerShell を実行するローカル コンピューターにファイルを保存します。たとえば C:\\Archive などです。

## 手順 2: ディレクトリ同期を構成する

ディレクトリ同期サービスでは、メール対応パブリック フォルダーは同期されません。次のスクリプトを実行すると、施設全体と Office 365 でメール対応パブリック フォルダーが同期されます。ハイブリッド展開シナリオではクロスプレミス アクセス許可がサポートされないため、メール対応パブリック フォルダーに割り当てられる特殊なアクセス許可をクラウド内で再作成する必要があります。詳しくは、「[Exchange Server 2013 のハイブリッド展開](exchange-server-hybrid-deployments-exchange-2013-help.md)」をご覧ください。


> [!NOTE]
> 同期されたメール対応パブリック フォルダーは、メール フロー用のメール連絡先オブジェクトとして認識されますが、EExchange 管理センター は表示できません。Get-MailPublicFolder コマンドを参照してください。クラウド内で SendAs アクセス許可を再作成するには、Add-RecipientPermission コマンドを使います。



1.  Exchange 2013 サーバーでは、次のコマンドを実行して、メール対応パブリック フォルダーをローカルのオンプレミス Active Directory から O365 に同期させます。
    
        Sync-MailPublicFolders.ps1 -Credential (Get-Credential) -CsvSummaryFile:sync_summary.csv
    
    ここで、`Credential` は Office 365 のユーザー名とパスワードで、`CsvSummaryFile` は同期操作やエラーを .CSV 形式で記録するためのパスです。


> [!NOTE]
> スクリプトを実行する前に、前述したように <CODE>-WhatIf</CODE> パラメーターを指定して実行することにより、スクリプトが環境内で実行するアクションをシミュレートすることをお勧めします。<BR>また、このスクリプトを毎日実行してメール対応パブリック フォルダーを同期させることもお勧めします。



## 手順 3: Exchange Online ユーザーが Exchange 2013 オンプレミス パブリック フォルダーにアクセスできるように構成する

この手順の最終ステップとして、Exchange Online 組織を構成して、Exchange 2013 パブリック フォルダーにアクセスできるようにします。

Exchange Online 組織からオンプレミスのパブリック フォルダーにアクセスできるようにします。オンプレミスのすべてのパブリック フォルダー メールボックスを指定することになります。

    Set-OrganizationConfig -PublicFoldersEnabled Remote -RemotePublicFolderMailboxes PFMailbox1,PFMailbox2,PFMailbox3


> [!NOTE]
> 変更を確認するには、Active Directory の同期が完了するまで待つ必要があります。この処理は、完了するまで最大 3 時間かかります。3 時間ごとに繰り返される同期処理を待機できない場合は、任意の時点でディレクトリの同期を強制実行できます。ディレクトリの同期を強制実行する詳細な手順については、「<A href="http://technet.microsoft.com/ja-jp/library/jj151771.aspx">ディレクトリを同期する</A>」を参照してください。



## 設定が適用されたことを確認する方法

1.  Exchange Online 内にいるユーザーの Outlook にログオンし、次のパブリック フォルダー テストを実行します。
    
      - 階層の表示
    
      - アクセス許可のチェック
    
      - パブリック フォルダーの作成と削除
    
      - パブリック フォルダーに対する内容の追加と削除


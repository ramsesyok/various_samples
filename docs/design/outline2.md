## システム構成

本システムは、3つのJavaアプリケーション（WARファイル）と、1つのC++製gRPCサービスで構成される。

### WAR構成

| WAR名 | 機能 | エンドポイント（URL） |
|-------|------|--------------------------|
| `user.war` | ユーザー管理画面とAPI | https://example.com/user/ |
| `product.war` | 製品管理画面とAPI | https://example.com/product/ |
| `api.war` | REST API（フロントなし） | https://example.com/api/ |

### gRPCバックエンド

- 計算用のバックエンドは C++ により実装され、gRPC を通じて `api.war` から呼び出される。
- 内部ネットワーク上で動作し、外部から直接アクセスされない。

```mermaid
graph TD
  A[Web Browser] --> B([https://example.com/user/])
  A --> C([https://example.com/product/])
  A --> D([https://example.com/api/])

  subgraph WebLogic Server
    B1[user.war]
    C1[product.war]
    D1[api.war]
  end

  B --> B1
  C --> C1
  D --> D1

  D1 --> E[gRPC C++ 計算サービス]
  D1 --> F[(DB)]

```
```
@startuml
skinparam componentStyle rectangle

actor User

node "WebLogic Server" {
  [user.war] --> [JSP/Servlet]
  [product.war] --> [JSP/Servlet]
  [api.war] --> [REST Controller]
}

cloud "Web Browser" {
  User --> [https://example.com/user/]
  User --> [https://example.com/product/]
}

User --> [https://example.com/api/] : REST API クライアント等

database "DB (Oracle等)" {
}

[api.war] --> [計算バックエンド (gRPC/C++)]
[api.war] --> database
@enduml
```
![alt text](ComponentDiagram2.png)

## JSP画面遷移図

以下はユーザー管理モジュールにおける、主な画面遷移の概要である。

- ユーザーは `/login` にアクセスして認証を行い、成功すれば `/mypage` に遷移する。
- `/mypage` では設定画面 `/settings` への遷移や、ログアウト処理 `/logout` を行う。

### 画面遷移図（PlantUML）
```mermaid
flowchart TD
  Start[開始] --> Login[ログイン画面]
  Login -->|ログイン成功| Mypage[マイページ]
  Login -->|失敗| Login
  Mypage --> Setting[設定画面]
  Mypage --> Logout[ログアウト確認]
  Setting --> Mypage
  Logout --> Login

```
```
@startuml
[*] --> ログイン画面 : アプリ起動

ログイン画面 --> マイページ : ログイン成功
ログイン画面 --> ログイン画面 : ログイン失敗

マイページ --> 設定画面 : 設定クリック
マイページ --> ログアウト確認画面 : ログアウトクリック
設定画面 --> マイページ : 戻る
ログアウト確認画面 --> ログイン画面 : 確認後ログアウト
@enduml

```
![alt text](JSPStateChart.png)


## URLパス構成図
以下は `user.war` 内の主要なパス構成である。

- `LoginController` → `/login`
- `MypageController` → `/mypage`
- `LogoutController` → `/logout`

### パス構成図
```
@startuml
package "user.war" {
  folder "Controller" {
    LoginController : /login
    MypageController : /mypage
    LogoutController : /logout
  }

  folder "JSP" {
    login.jsp
    mypage.jsp
    logout_confirm.jsp
  }

  LoginController --> login.jsp
  MypageController --> mypage.jsp
  LogoutController --> logout_confirm.jsp
}
@enduml
```
![alt text](PathDiagram.png)


```mermaid
graph TD
  root[/user/]
  root --> login[ /login ]
  root --> mypage[ /mypage ]
  root --> logout[ /logout ]

  login --> loginjsp[login.jsp]
  mypage --> mypagejsp[mypage.jsp]
  logout --> logoutjsp[logout_confirm.jsp]

```
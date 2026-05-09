# 三大模块依赖关系图谱 (Mermaid)

> 可用支持 Mermaid 的工具（Typora、Obsidian、GitHub、Notion）打开此文件查看可视化图表。

---

## 1. 全局架构依赖图

```mermaid
graph TB
    subgraph Infrastructure["Infrastructure Layer"]
        Router[ESmartRouter]
        UserManager[ESmartUserManager]
        HouseData[ESHouseDataManager]
        ApiService[ESmartApiServiceBase]
        CTMediator[CTMediator]
        ConfigMgr[SmartHongXinNetWorkManager]
        NotifMgr[SmartNotificationManager]
        ReportMgr[ESReportManager]
        Cache[CacheTools]
        MineSetting[ESmartMineSettingService]
    end

    subgraph Gateway["Gateway 网关模块 (533 files)"]
        GW_Target[TargetGateway]
        GW_Service[ESmartGatewayService]
        GW_Ability[NSmartNetAbilityServices<br/>NSmartNetBaseServices]
        GW_VM[ESmartGatewayViewModel]
        GW_Model[ESmartGatewayModel]
        GW_UI[Controllers + Views]
        GW_SDK[TelecomStewardSDK<br/>SDN SDK]
        GW_Target --> GW_Service
        GW_Service --> GW_Ability
        GW_Ability --> GW_SDK
        GW_Service --> GW_VM
        GW_VM --> GW_Model
        GW_Model --> GW_UI
    end

    subgraph HomePage["HomePage 首页模块 (310 files)"]
        HP_Target[TargetHome]
        HP_VM5[ESmartHomePageViewModel<br/>5.x]
        HP_VM6[ESHomePageVM<br/>6.0]
        HP_Model[ESmartCardModel]
        HP_Repo[ESmartCardDataRepsitory<br/>ESHomePageDataRepsitory]
        HP_UI5[ESmartHomePageViewController<br/>+ Card Views]
        HP_UI6[ESHomePageVC<br/>+ Device/Room/Service Tabs]
        HP_Net[ESmartHomeRequest]
        HP_Popup[HomePagePopupTools<br/>HomePagePopupManager]
        HP_Target --> HP_VM5
        HP_VM6 --> HP_Repo
        HP_VM5 --> HP_Model
        HP_VM6 --> HP_Model
        HP_Model --> HP_UI5
        HP_Model --> HP_UI6
        HP_VM5 --> HP_Net
        HP_VM6 --> HP_Net
        HP_Popup --> HP_UI5
        HP_Popup --> HP_UI6
    end

    subgraph Message["MessageCenter 消息模块 (105 files)"]
        MC_Target[TargetMessage]
        MC_Mgr[ESmartMessageManager<br/>singleton + FMDB]
        MC_VM[MCMessagesViewModel<br/>MCSettingViewModel]
        MC_Model[MCMessageRecord<br/>MCMsgConfigModel]
        MC_UI[MCMainViewController<br/>MCMsgListViewController<br/>MCSettingViewController]
        MC_Push[ESmartPush / ESmartPushSDK]
        MC_Wechat[ESmartWechatPublicService<br/>+ WechatOpenSDK]
        MC_Target --> MC_Mgr
        MC_Mgr --> MC_VM
        MC_VM --> MC_Model
        MC_VM --> MC_UI
        MC_Mgr --> MC_Push
        MC_Mgr --> MC_Wechat
    end

    Router -.-> GW_Target
    Router -.-> HP_Target
    Router -.-> MC_Target
    UserManager -.-> GW_Service
    UserManager -.-> HP_VM5
    UserManager -.-> MC_Mgr
    HouseData -.-> GW_Service
    HouseData -.-> HP_VM5
    HouseData -.-> MC_VM
    ApiService -.-> GW_Service
    ApiService -.-> HP_Net
    ApiService -.-> MC_Mgr
    ConfigMgr -.-> GW_VM
    ConfigMgr -.-> HP_VM6
    ConfigMgr -.-> MC_UI
    NotifMgr -.-> GW_Service
    NotifMgr -.-> HP_VM5
    NotifMgr -.-> MC_Mgr

    GW_Service -. "kGatewayRebootNotification" .-> HP_VM5
    GW_UI -. "ESmartSmartHomeReloadDeviceList" .-> HP_VM6

    MC_Mgr -. "kNotiUpdateMsgIcon" .-> HP_UI5
    MC_Mgr -. "kNotiStartMsgIconAnimation" .-> HP_UI5

    HP_Target -. "actionBindFindGateway" .-> GW_Target
    HP_UI5 -. "ESmartGatewayService" .-> GW_Service
    HP_UI6 -. "ESmartGatewayService+Platform" .-> GW_Service
```

---

## 2. Gateway 模块内部数据流

```mermaid
graph TB
    Mediator[CTMediator<br/>TargetGateway 12+ routes]

    GWFacade[ESmartGatewayService<br/>Singleton 1590 lines]
    Platform[ESmartGatewayService+Platform<br/>REST APIs]
    HomeNet[ESmartGatewayService+HomeNet<br/>Direct-send APIs]
    Ability[NSmartNetAbilityServices<br/>1596 lines]
    BaseService[NSmartNetBaseServices<br/>1954 line header]
    SpeedHelper[SpeedHelper<br/>YYTimer polling]

    TelecomSteward[TelecomStewardSDK]
    SDNSDK[SDNManager / SDNTools]

    GWVM[ESmartGatewayViewModel<br/>NSOperationQueue + Semaphore]

    DeviceModel[NSmartNetDeviceModel]
    GWModel[ESmartGatewayModel<br/>+ 20 sub-models]
    FuncModel[FunctionModel<br/>23 function types]

    GWHomeVC[ESmartGatewayHomeBaseController<br/>UICollectionView]
    GWBindVC[TYGateWayBindController]
    GWSettingVC[WiFi/Gateway Setting VC]
    GWSpeedVC[TYGatewaySpeedTestVC]
    GWTopoVC[ESGatewayTopologyController]

    Mediator --> GWFacade
    GWFacade --> Platform
    GWFacade --> HomeNet
    GWFacade --> Ability
    Ability --> BaseService
    Ability --> TelecomSteward
    Ability --> SDNSDK
    BaseService --> TelecomSteward

    GWFacade --> GWVM
    GWVM --> GWModel
    GWVM --> SpeedHelper
    GWModel --> DeviceModel
    GWModel --> FuncModel

    GWVM --> GWHomeVC
    GWFacade --> GWBindVC
    GWFacade --> GWSettingVC
    GWFacade --> GWSpeedVC
    GWFacade --> GWTopoVC
```

---

## 3. HomePage 6.0 模块内部数据流

```mermaid
graph TB
    ESHPVC[ESHomePageVC<br/>IGListAdapter + Nested Scroll]

    ESHPVM[ESHomePageVM<br/>Main Aggregator]
    SearchVM[ESHomePageSearchVM]
    ToolbarVM[ESHomePageToolBarVM]
    AppVM[ESHomePageAppVM]
    NetworkVM[ESHomePageNetworkVM]
    TabsVM[ESHomePageTabsVM<br/>Device / Room / Service]

    DeviceVM[ESHomePageDeviceVM]
    RoomVM[ESHomePageRoomVM]
    ServiceVM[ESHomePageServiceVM]
    CloudVM[ESHomePageCloudVM]
    SceneVM[ESHomePageSceneVM]
    DailyVM[ESHomePageDailyVM]
    CommunityVM[ESHomePageCommunityVM]
    VillageVM[ESHomePageVillageVM]

    Repo[ESHomePageDataRepsitory<br/>Template + Cache]

    Ext1[SmartHongXinNetWorkManager]
    Ext2[ESmartGatewayService+Platform]
    Ext3[ESmartSmartHomeFamilyViewModel]
    Ext4[ESmartSmartHomeRoomViewModel]
    Ext5[ESmartMineSettingService]
    Ext6[ESmartMessageManager]
    Ext7[ESCommunityService]
    Ext8[ESHouseDataManager]

    ESHPVC --> ESHPVM
    ESHPVM --> SearchVM
    ESHPVM --> ToolbarVM
    ESHPVM --> AppVM
    ESHPVM --> NetworkVM
    ESHPVM --> TabsVM
    TabsVM --> DeviceVM
    TabsVM --> RoomVM
    TabsVM --> ServiceVM
    ServiceVM --> CloudVM
    ServiceVM --> SceneVM
    ServiceVM --> DailyVM
    ServiceVM --> CommunityVM
    ServiceVM --> VillageVM

    Repo --> ESHPVM
    Ext1 --> Repo
    Ext2 --> NetworkVM
    Ext3 --> DeviceVM
    Ext4 --> DeviceVM
    Ext5 --> DeviceVM
    Ext6 --> ToolbarVM
    Ext7 --> CommunityVM
    Ext7 --> VillageVM
    Ext8 --> DeviceVM
```

---

## 4. MessageCenter 模块内部数据流

```mermaid
graph TB
    Mediator[CTMediator<br/>TargetMessage 10 routes]
    Push[前台推送]

    MsgMgr[ESmartMessageManager<br/>singleton 2100+ lines]
    H5Bridge[MCH5SettingBridge]

    FMDB[FMDB SQLite<br/>{tyAccount_md5}_smart_message.db]

    BindToken[bind_token / unbind_token]
    GetConfig[get_user_config]
    GetMsgList[record_page_list]
    DeviceConfig[get/set_device_config]
    DNDConfig[get/set_disturb_config]
    WechatAPI[WeChat APIs]

    MsgVM[MCMessagesViewModel<br/>网络→FMDB→UI转换]
    SettingVM[MCSettingViewModel<br/>设置页数据工厂]

    MainVC[MCMainViewController]
    ListVC[MCMsgListViewController]
    EditVC[MCMsgEditViewController]
    SettingVC[MCSettingViewController]
    WechatVC[MCWeChatPublicInfoViewController]

    TPNS[TPNS SDK]
    Baichuan[百川 SDK]

    Mediator --> MsgMgr
    Push --> MsgMgr
    MsgMgr --> FMDB
    MsgMgr --> BindToken
    MsgMgr --> GetConfig
    MsgMgr --> GetMsgList
    MsgMgr --> DeviceConfig
    MsgMgr --> DNDConfig
    MsgMgr --> WechatAPI
    MsgMgr --> TPNS
    MsgMgr --> Baichuan

    FMDB --> MsgVM
    BindToken --> MsgVM
    GetConfig --> MsgVM
    GetMsgList --> MsgVM
    DeviceConfig --> SettingVM
    DNDConfig --> SettingVM
    WechatAPI --> WechatVC

    MsgVM --> ListVC
    MsgVM --> MainVC
    SettingVM --> SettingVC
    H5Bridge --> SettingVC
    H5Bridge --> MainVC
    MsgMgr --> EditVC
```

---

## 5. 跨模块通知依赖图

```mermaid
graph LR
    P1[ESHouseDataManager]
    P2[ESmartHomePageViewModel]
    P3[ESmartMessageManager]
    P4[Gateway Controllers]
    P5[ESmartGatewayService]

    N1[ESmartDidChangeDefaultGateway]
    N2[ESmartDidRefreshDefaultGateway]
    N3[kNotiUpdateMsgIcon]
    N4[kNotiStartMsgIconAnimation]
    N5[ESmartSmartHomeReloadDeviceList]
    N6[kGatewayRebootNotification]
    N7[SMARTCHANGEUSERAREA]
    N8[SMARTACCOUNTLOGOUT]

    S1[NSmartNetAbilityServices]
    S2[ESmartHomePageViewModel]
    S3[ESmartHomePageViewController]
    S4[ESHomePageDataRepsitory]
    S5[Gateway Controllers]
    S6[外部监听者]

    P1 -->|发布| N1
    P2 -->|发布| N2
    P3 -->|发布| N3
    P3 -->|发布| N4
    P4 -->|发布| N5
    P5 -->|发布| N6

    N1 -->|订阅| S1
    N1 -->|订阅| S2
    N2 -->|订阅| S1
    N3 -->|订阅| S3
    N4 -->|订阅| S3
    N5 -->|订阅| S4
    N6 -->|订阅| S6
    N7 -->|订阅| S2
    N7 -->|订阅| S1
    N8 -->|订阅| S2
    N8 -->|订阅| S1
```

---

## 6. 耦合节点风险热力图

```mermaid
graph TB
    subgraph HIGH["🔴 高危耦合 (4个节点)"]
        H1[ESmartRouter<br/>全局路由中心<br/>3模块共用]
        H2[ESmartUserManager<br/>用户状态单例<br/>3模块共用]
        H3[ESHouseDataManager<br/>家庭数据中枢<br/>3模块共用]
        H4[MessageManager → HomePage<br/>消息图标通知硬耦合]
    end

    subgraph MEDIUM["🟡 中危耦合 (4个节点)"]
        M1[SmartHongXinNetWorkManager<br/>配置中心 3模块共用]
        M2[NSmartNetAbilityServices<br/>Gateway→HomePage间接依赖]
        M3[网关操作 → 首页刷新<br/>ESmartSmartHomeReloadDeviceList]
        M4[ESmartGatewayService<br/>1590行单例门面]
    end

    subgraph LOW["🟢 低危耦合 (3个节点)"]
        L1[ESmartMineSettingService<br/>设置服务]
        L2[ESmartCommunityAndVillageService<br/>社区村镇服务]
        L3[CTMediator<br/>组件化路由 框架级]
    end

    H1 --> M1
    H2 --> M2
    H3 --> M3
    H4 --> M4
    M1 --> L1
    M2 --> L2
    M3 --> L3
```

---

## 7. 拆分路线图

```mermaid
gantt
    title 三大模块拆分实施路线图
    dateFormat  YYYY-MM-DD
    axisFormat  %m/%d

    section P0 紧急解耦
    解耦 MessageManager→HomePage 通知耦合   :p0_1, 2026-05-11, 5d
    统一 ESmartRouter 路由常量管理          :p0_2, 2026-05-11, 3d

    section P1 核心拆分
    拆分 Gateway → Core/UI/Bind           :p1_1, 2026-05-16, 10d
    统一 ESHouseDataManager 响应式数据流    :p1_2, 2026-05-16, 7d

    section P2 首页拆分
    拆分 HomePage → Core/Cards/Tabs       :p2_1, 2026-05-26, 10d
    收敛 ESmartGatewayService 公开API       :p2_2, 2026-05-26, 3d

    section P3 收尾
    拆分 MessageCenter 子模块              :p3_1, 2026-06-05, 5d
    统一用户状态事件总线                   :p3_2, 2026-06-05, 7d
```

---

*使用支持 Mermaid 的编辑器打开此文件可渲染为交互式图表。*

# dac

## 简介
```shell
    1. 支持重定向的主动设备代表为新协议的 Ehome 设备，
       不支持重定向协议的主动设备代表为老协议的 Ehome 设备（2.0协议，4.0协议均不支持）
    2. 不支持重定向协议的主动设备只能在设备上配置接入服务的地址信息，一旦配置，无法动态修改，只能登陆设备进行修改，
      因此不支持重定向协议的主动设备不支持集群方式接入, 重定向协议的设备先向重定向服务进行注册，重定向服务获取接入服务节点
      的状态信息，选择最优的服务节点，并将其信息发送给重定向设备,之后重定向设备在向目标服务器节点进行注册.
    3. das/ldm -> 设备接入框架总体设计说明书
    
        (1)本地驱动管理器(ldm/das)
            ConfigMgr -> 配置管理模块
            DevResMgr -> 设备资源管理模块
            DriverMgr -> 驱动管理模块
            ProtocolProxy -> 协议代理模块
            LdmService -> 主服务模块 
            
            a. 配置管理模块
                主要通过 dac.1/conf/installation.properties(安装配置文件) 和　
                dac.1/conf/config.properties(服务配置文件) 获取自身服务信息和设备管理服务信息，
                提供上述信息给本地驱动管理器的其他模块使用
                
            b. 协议代理模块
                1. 设备资源管理类接口 
                        资源变更通知, 获取设备详细信息, 根据编号查询设备信息, 获取驱动管理的设备列表, 下发临时设备信息
                2. 驱动管理类接口 
                        获取驱动信息, 查询驱动实例信息, 获取驱动实例进程, 按资源获取驱动实例, 驱动启用, 驱动禁用
                        重启驱动, 获取驱动配置, 设置驱动配置, 获取驱动运行状态, 获取驱动实例状态, 获取驱动进程状态
                        获取驱动进程监控指标, 获取设备模型, 获取驱动多语言资源
                3. 服务管理类接口
                        获取本地驱动管理器运行状态, 线路变更
                4. 安全类接口
                        密钥交换接口
                5. 协议代理类接口，将请求消息转发到驱动，并将驱动返回的结果返回给请求者
                        透明传输
                        
            c. 设备资源管理模块
                设备资源管理模块主要功能向设备管理服务(DMS)注册并同步资源, 进行设备资源内容转换后将设备资源下发给驱动管理模块
                (驱动管理模块再将设备分配到驱动实例中), 同时提供基本的查询能力，如根据资源编号查询所属设备编号,
                根据主动设备编号查询设备编号等
                
            d. 驱动管理模块
                1. 读取驱动的驱动描述文件，并将信息保存到内存中
                2. 实现驱动启用、禁用，驱动实例查询，驱动实例运行状态查询接口
                3. 根据设备创建驱动实例，向驱动实例下发设备信息
                4. 管理被动设备发现服务和主动设备注册服务
                5. 代理设备发现、设备信息远程获取、设备操作
                6. 驱动管理模块与驱动之间通过统一的驱动网络库(DrvNetLib)完成交互
                        1） 设备资源管理类接口
                               同步设备, 增加设备 ,删除设备, 修改设备；
                        2） 驱动管理类接口
                                驱动实例注册, 驱动实例心跳；
                        3） 设备操作类接口
                                透明传输；
                                
            e. 主服务模块
                    加载配置管理模块、协议代理模块、设备资源管理模块、驱动管理模块，完成上述各模块的初始化、启动、以及参数和消息传递。
                    提供符合 hservice 标准的接口，通过 hservice 加载，将本地驱动管理器安装成服务



            
        (2) 媒体转分发程序(das_media) dac.1/bin/ldm/apps/das_media_linux/stream/das_media 
                das_media_linux
                    das_media_cfg
                    plugins
                        dahua_plugin
                        ehome_plugin
                        hik_plugin
                        onvif_plugin
                        private_stream_plugin
                        rtsp_plugin
                    stream
                        
        (3) Ehome代理程序(ehome_reg_svr/RegService)  dac.1/bin/ldm/drivers/reg_ehome_register_svr
                a. 采用 Ehome 代理程序场景:
                        接入设备的驱动存在多个驱动程序，如 Ehome 视频门禁一体机驱动，其驱动中包含视频能力驱动程序、门禁能力驱动进程。
                        如果按照当前的实现方式，由驱动直接接入主动设备，会出现主动注册端口需要开放多个，以及同一款设备需要向多个端口注册的问题
                        
                b. 解决方案:
                        对设备提供统一的主动设备注册服务(Ehome代理程序)，所有主动设备都向其进行注册，
                        根据从本地驱动管理器获取的驱动实例信息，完成设备与驱动程序之间协议的代理转发
            
        
    4. 本地驱动管理器(ldm)对上接口: << 设备接入框架（本地驱动管理器）接口说明书-s2 >> 
    5. 本地驱动管理器(ldm)对下接口: << 设备接入框架与设备驱动间通讯协议 >>
    6. 供其他组件调用　"设备接入框架"　-> 《设备接入框架接口说明书.pdf》(对 DMS 通讯)
    7. 供其他驱动调用 "设备接入框架" ->  << 设备接入框架与设备驱动间通讯协议 >>, 能力协议 -> 状态 -> << 搜索发现能力协议.pdf >> 
       
```

## 接入驱动

```shell
    1. 驱动文件目录结构(成果物)
            (1) conf 文件夹存放驱动配置文件(对应与 Hido 上的 [驱动参数设置])
            (2) language 文件夹用于存放语言包文件夹，语言包文件夹包含内翻译文件 translate.properties，更新日志changelog.txt, 
                设备支持信息 device_support.txt, 功能描述文件 function_description.txt, 
                性能描述文件 performance_description.txt
            (3) META-INF 文件夹存放驱动描述文件 driver.xml(包含了 Hido 中的[驱动描述]), 驱动维护信息文件 maintenance.xml, 
                描述文件校验文件 file_checksum.xml，程序文件校验文件 program_checksum.xml 
            (4) model/fireprotection_fireDevice_model_1.2.0.xml : 设备模型
```

## dac 驱动的组成

```shell
    1. 
        
    2. 类似的该打包驱动有 
            例如　drv_vss_onvif_general_1.10.101　　ONVIF协议视频驱动
                    (1) 先由 dagserver_vss 工程生成的 libDagServer.so
                    (2) drivers->devplugins->VagAdapterDev 工程生成的 libVagAdapterDev.so
                    (3) drivers->devplugins->onvif_plugin 工程生成的 libonvif_plugin.so
                    
            a. 
                drv_vss_onvif_general(ONVIF协议视频驱动)  
                package_drv_vss_onvif.sh 
                makefile_onvif_plugin
                onvif_plugin 工程
            b.
                drv_vss_gb_general(GB/T28181视频驱动)  
                package_drv_vss_gb.sh
                makefile_gb28181_plugin
                gb28181_plugin　工程
            c.
                drv_vss_ehome_general(eHome协议视频驱动)　
                package_drv_vss_ehome.sh
                makefile_ehome_plugin
                ehome_plugin　工程
            d.
                drv_vss_dhsdk_general(大华私有协议视频驱动)　
                package_drv_vss_dahua.sh
                makefile_dahua_plugin
                dahua_plugin 工程
                
            e.
                drv_emer_ehome(eHome协议紧急报警驱动)
                package_drv_emer_ehome.sh
                makefile_ehome_plugin
                ehome_plugin　工程
                
    3. drv_emer_hiksdk_allinone(海康私有协议紧急报警设备一体机驱动)
            这是个复合驱动, package_drv_emer_hiksdk_allinone.sh, 用了[dagserver_vss 工程]生成的 libDagServer.so，
            使用　makefile_HikDev ([HikDev 工程]) 生成 libHikDev.so  , 使用 makefile_HikDevSearch 
            [HikDevSearch　工程] 生成 libHikDevSearch.so
            
            复合驱动中 emer(紧急报警进程)和 vag(视频)都有以上的动态链接库, 但是 配置文件不同(DAGConfig.xml), emer 里面多了
             <!--驱动类型: AlarmHostDrv, HikDrv, EhomeDrv, ... -->
             <DrvType>AlarmColumnDrv</DrvType>
             
    4. drv_ias_hiksdk(海康私有协议报警主机驱动)
            (1) 先由 dagserver_vss 工程生成的 libDagServer.so
            (2) makefile_HikDev ([HikDev 工程]) 生成 libHikDev.so
            (3) 使用 makefile_HikDevSearch [HikDevSearch　工程] 生成 libHikDevSearch.so
            package_drv_ias_hiksdk.sh
            
    5. drv_int_hiksdk(海康私有协议智能驱动)
            (1) 先由 dagserver_vss 工程生成的 libDagServer.so
            (2) makefile_HikDev ([HikDev 工程]) 生成 libHikDev.so
            (3) 使用 makefile_HikDevSearch [HikDevSearch　工程] 生成 libHikDevSearch.so
            package_drv_int_hiksdk.sh
            
    6. drv_vss_hiksdk_general(海康私有协议视频驱动)
            (1) 先由 dagserver_vss 工程生成的 libDagServer.so
            (2) makefile_HikDev ([HikDev 工程]) 生成 libHikDev.so
            (3) 使用 makefile_HikDevSearch [HikDevSearch　工程] 生成 libHikDevSearch.so
            package_drv_vss_hiksdk.sh
```

## dac 库使用

```shell
    1. drv_agent_server
            (1) xml 的封装库　DagXmlHelper.h　
                    TiXmlUtils::to_number(CDagXmlHelper::GetXmlDataStr(pRootElement, "PtzConfig.MinPtzCtrlTimeout",
                    to_string<int>(DEFAULT_MIN_PTZ_CTRL_TIMEOUT)).c_str(), m_ptzConfig.minPtzCtrlTimeout);
                    
                    int CDagServerConfig::LoadXMLData()
                    
            (2) 原子变量的使用
                    CHikAtomicOperationT<CHikLockMutex> m_nMgrDevCount;    // 所有设备数量
                    m_nMgrDevCount.GetValue();
                    m_nMgrDevCount++;
                    m_nMgrDevCount--;
                    
            (3) HikTimeValue.h 保存时间值
                    m_htvInterval.Set(30);  // 设置 30 秒
                    
            (4) IHikThreadManager 线程的使用
                    第一步: 创建线程 IHikThread* m_pThread, 这个线程创建后有一个独立线程环境运行, 具有事件队列、时钟队列和事件多路分离器
                            IHikThreadManager::Instance()->CreateThread(FALSE, IHikThreadManager::HIK_THREAD_EVENT_QUEUE_TIMER_QUEUE_REACTOR, NULL, m_pThread);
                            
                    第二步 : 创建一个派生类 CDagScheduleTimerEvent 继承 IHikCustomizeEvent, 重载了 IHikCustomizeEvent::OnEvent()
                            派生类 CDagScheduleTimerEvent 可以定义私有变量, 保存自身的要调用的信息
                    第三步: 投递事件, 让　m_pThread 触发运行
                                result = m_pThread->SendEvent(pEvent)  // 阻塞式投递, 运行完事件后, SendEvent 才返回
                                result =　m_pThread->PostEvent(pEvent)　　// 非阻塞投递, PostEvent 立马返回
                                
                                这个时候就会触发覆盖写的 CDagScheduleTimerEvent::OnEvent()
                                
                                
            (5) CHikUtilThreadPool 线程池的使用
                    a. 对外使用
                        第一步: CHikUtilThreadPool 对象的 CHikUtilThreadPool::init();
                        第二步: 开始投递事件 CHikUtilThreadPool::PostTaskEvent(IHikCustomizeEvent* pEvent)
                               其中传入参数是 HikCustomizeEvent 派生类,需要重写 OnEvent()
                               
                    b. 内部实现
                            1. 自身类内创建了一个私有的线程 IHikThread* m_pThreadPool, 主要是用来对　PostTaskEvent() 函数
                            　　传进来的　HikCustomizeEvent 派生类参数进行封装，再产生一个事件对象 ThreadPoolEvent
                               用于 m_pThreadPool 投递, 同时覆写 OnEvent(), OnEvent() 里面的操作是
                                    (1) 如果有空闲的 thread 是使用,则从 freeThreads 拿来一个空闲的 thread 来使用投递, 
                                            投递成功后(基本上代表事件肯定会被调用, 其保存到内部的数组中), 再从 task 取出事件进行处理,
                                            直至处理完归还给 freeThreads 中.
                                    (2) 如果没有空闲的 thread, 则直接保存到 task 队列中.
                                    
            (6) 事件码的时间生成
                    // 获取本地时区
                                char chTimeZone[32] = {0};
                                // 更新脉冲事件最新时间;
                                char szLocalHappenTime[64] = {0};
                                HPR_TIME_EXP_T stLocalHappenTime = {0};
                                HPR_TIME_T nowTime = HPR_TimeNow();
                                HPR_ExpTimeFromTimeLocal(nowTime, &stLocalHappenTime);
                    
                                GetLocalTimeZone(chTimeZone);     // 获取夏令时时区
                                HPR_Snprintf(szLocalHappenTime, sizeof(szLocalHappenTime) - 1, "%d-%02d-%02dT%02d:%02d:%02d.%03d%s",
                                             stLocalHappenTime.tm_year + 1900, stLocalHappenTime.tm_mon + 1, stLocalHappenTime.tm_mday,
                                             stLocalHappenTime.tm_hour, stLocalHappenTime.tm_min, stLocalHappenTime.tm_sec, stLocalHappenTime.tm_usec / 1000,
                                             chTimeZone);
                    
                                string sNowTime = szLocalHappenTime;
                                
            (7) dag_server 处理 ldm 下发命令的流向
                    CDrvNetTransMgr::OnReceiveData() ->  m_cTaskScheduler.addTask(pTask) ->  CTaskScheduler::insideThreadRoutine() -> 
                    CTask::Execute() -> CTask::CreateObj(string method, Json::Value& jsonValue, bool bTrans) -> 
                    CBaseTask::Execute()
                    {
                        Run() // 继承类 Run() 重载
                        
                        if (m_rHeader.GetSubType() == DEVICE_INPUTDEVICELIST_REQ || m_rHeader.GetSubType() == DEVICE_INPUTDEVICELISTEX_REQ)
                        {
                            CDagDeviceMgrSingleton::Instance()->ProcDeviceResReq(m_apClient, &m_rHeader, pmbData.Get());
                        }
                        else
                        {
                            CDagCtrlManagerSingleton::Instance()->ProcDeviceCtrlRequest(m_apClient, &m_rHeader, pmbData.Get());
                        }
                    }
                    
                     
            (8) 设备抓图
                    void CTaskScheduler::insideThreadRoutine()
                    {
                     pNewTask->Execute();
                    }
                    
                    // 如果继承类重载了 Execute , 则调用特有的 Execute, 否则调用基类的 Execute
                    int CTask::Execute()
                    {
                     int retValue = CTaskFactorySingleton::Instance()->CreateObj(method, pTask, bTrans);
                      pTask->Execute();
                    }
                    
                    class CCapturePictureTask :public CBaseTask
                    {
                    public:
                        CCapturePictureTask();
                        virtual int Run();  // 进行数据赋值
                    };
                    
                    // 如果继承类重载了 Execute , 则调用特有的 Execute, 否则调用基类的 Execute
                    int CBaseTask::Execute()
                    {
                     int iRet = Run();
                     CHikMessageBlockAutoPtr pmbData;
                     DagProtobufSerializeToMB(m_pMessage, pmbData );
                    
                     if (m_rHeader.GetSubType() == DEVICE_INPUTDEVICELIST_REQ || m_rHeader.GetSubType() == DEVICE_INPUTDEVICELISTEX_REQ)
                     {
                     CDagDeviceMgrSingleton::Instance()->ProcDeviceResReq(m_apClient, &m_rHeader, pmbData.Get());
                     }
                     else
                     {
                     CDagCtrlManagerSingleton::Instance()->ProcDeviceCtrlRequest(m_apClient, &m_rHeader, pmbData.Get());
                     }
                    }
                    
                    int CDagCtrlManager::ProcDeviceCtrlRequest(CDagClientAutoPtr& apClient,  CDagMessageHeader* pmhHeader, CHikMessageBlock* pmbData)
                    {
                        map<int, IDagCtrlReqProcInf*>::iterator itr = m_mapClientReqProcObjectList.find(nRequestType);  // 找到相应的派生类
                     int nRet = itr->second->ProcDevCtrlReq(apClient, pmhHeader, pmbData);
                    }
                    
                    
                    class CCapturePictureExObject:public IDagCtrlReqProcInf
                    {
                    public:
                        virtual int ProcDevCtrlReq(CDagClientAutoPtr&  apClient, CDagMessageHeader* pmhHeader, CHikMessageBlock* pmbData);
                        virtual int ExcuteDevCtrlCmd(CDagClientAutoPtr&  apClient, CDagMessageHeader* pmhHeader, vector<CDagDeviceAutoPtr>& vecDevList, Message* pRequest);
                    };
                    
                    
                    // 抓图功能
                    int CCapturePictureExObject::ExcuteDevCtrlCmd( CDagClientAutoPtr& apClient, CDagMessageHeader* pmhHeader, vector<CDagDeviceAutoPtr>& vecDevList, Message* pRequest )
                            

```

## tinmy

```shell
    1. 解析 xml
       
       (1)
        <?xml version="1.0" encoding="UTF-8" ?>
        <drivers>
            <reg_isup5_register_svr_1.10.100 status="enable" />
        </drivers>
        
        TiXmlDocument oXml(文件名);
        if (!oXml.LoadFile())
        {
            DAF_WARN("[0x%08x] - [errorId=%d,errorDesc=%s] load driver status fail", ERROR_CODE_OPEN_FILE_FAIL, oXml.ErrorId(),oXml.ErrorDesc());
            return ERROR_CODE_PARSE_FAIL;
        }
    
        const TiXmlElement* pRoot = oXml.RootElement();//Configuration
        if (NULL == pRoot)
        {
            DAF_WARN("[0x%08x] - parse config fail, no root", ERROR_CODE_PARSE_FAIL);
            return ERROR_CODE_PARSE_FAIL;
        }
    
        const TiXmlElement* pDrvSts = pRoot->FirstChildElement();
        for (; pDrvSts != NULL; pDrvSts = pDrvSts->NextSiblingElement())
        {
            const char* pszSts = pDrvSts->Attribute("status"); // enable
            if (pszSts == NULL)
            {
                continue;
            }
            mapStatus[pDrvSts->ValueStr()] = pszSts; // pDrvSts->ValueStr() : reg_isup5_register_svr_1.10.100
        }
        
    2. 设置 xml
    
        (1)
            <drivers>
                <reg_isup5_register_svr_1.10.100 status="enable" />
            </drivers>
            
            TiXmlDocument oXml(文件名);
            if (!oXml.LoadFile())
            {
                DAF_WARN("[0x%08x] - [errorId=%d,errorDesc=%s]load driver status fail", ERROR_CODE_OPEN_FILE_FAIL, oXml.ErrorId(), oXml.ErrorDesc());
                return ERROR_CODE_OPEN_FILE_FAIL;
            }
        
            TiXmlElement* pRoot = oXml.RootElement();//Configuration
            TiXmlElement* pDrvSts = pRoot->FirstChildElement(itr->first);
            if (pDrvSts == NULL)  // 没有找到, 新建一个节点
            {
                TiXmlElement* pNewNode = new(std::nothrow) TiXmlElement(itr->first);
                if (pNewNode == NULL)
                {
                    DAF_ERROR("[0x%08x] - [drvId=%s,status=%s]add driver status fail, new mem fail",
                        ERROR_CODE_NEW_FAILED, itr->first.c_str(), itr->second.c_str());
                    continue;
                }
                pNewNode->SetAttribute("status", itr->second.c_str());
    
                if (NULL == pRoot->LinkEndChild(pNewNode))
                {
                    DAF_WARN("[0x%08x] - [drvId=%s,status=%s,errorDesc=%s]add driver status fail",  
                        ERROR_CODE_PARAM_ERROR, itr->first.c_str(), itr->second.c_str() ,oXml.ErrorDesc());
                    continue;
                }
            }
            else  // 找到直接设置
            {
                pDrvSts->SetAttribute("status", itr->second.c_str());
                DAF_INFO("- [drvId=%s,status=%s]set driver status succ", itr->first.c_str(), itr->second.c_str());
            }
            
            if (oXml.SaveFile())
            {
                DAF_INFO("- save driver_status succ");
                iRet = ERROR_CODE_SUCCESS;
            }
            else
            {
                DAF_ERROR("[0x%08x] - [errorId=%s,errorDesc=%s]save driver_status fail", ERROR_CODE_SAVE_FILE_FAIL, oXml.ErrorId(),oXml.ErrorDesc());
                iRet = ERROR_CODE_SAVE_FILE_FAIL;
            }
```

## ldm
```shell
    1. DevResMgr 模块(设备资源管理模块)
            DevResMgr.h -> DeviceMgr.h -> Device.h 和 DataSrcMgr.h
    2. DriverMgr 模块(驱动管理)
            DriverCIf.h -> DriverMgr.h
```

## 库使用
```shell
    1. HCNetUtils库 HCNetUtils库由本公司开发的通信协议库，用于支持HTTP1.1协议，web socket(版本13)协议，以及SMTP的客户端协议
    2.  HikThreadInterface.h 中的 
            class IHikThread;
            class IHikThreadManager;
            class IHikThreadSink;
            class IHikCustomizeEvent;
            这些类可以配合使用
            
                  IHikThread* m_pPreConnTimerThread;  // 重连时钟线程
                  IHikThreadManager::Instance()->CreateThread(FALSE, IHikThreadManager::HIK_THREAD_EVENT_QUEUE_TIMER_QUEUE_REACTOR, 
                  　　　　　　　　　　　　　　　　　　　　　　　　　　this, m_pPreConnTimerThread);
            
                  CDevMgrUnInitDevEvent* pUnInitEvent = new(std::nothrow) CDevMgrUnInitDevEvent(itr->second, this);
                  m_pPreConnTimerThread->PostEvent(pUnInitEvent);  // 非阻塞
                   m_pPreConnTimerThread->SendEvent(pEvent); // 阻塞
                   
                   这个时候 CDevMgrUnInitDevEvent::OnEvent() 会被调用
                   
    3. HRP_AsyncIOEX.h(创建异步 IO 队列)
    
            (1)  HPR_HANDLE m_hAsyncIOQueue = HPR_AsyncIO_CreateQueueEx_New(8);  // 输入参数线程数为 8 
            (2) 销毁异步 IO 队列
                HPR_AsyncIO_DestroyQueueEx(m_hAsyncIOQueue);
                m_hAsyncIOQueue = HPR_INVALID_ASYNCIOQUEUE;
                
            (3) 
                // 将已设置好的 m_iConnSocketFd 与 已初始化好的 m_hAsyncIocpQueue 进行绑定.
                iRet = HPR_AsyncIO_BindIOHandleToQueueEx((HPR_HANDLE)m_iConnSocketFd, m_hAsyncIocpQueue);
                // 将已设置好的 m_iConnSocketFd 与 设置好的回调函数绑定
                iRet = HPR_AsyncIO_BindCallBackToIOHandleEx((HPR_HANDLE)m_iConnSocketFd, pfnIocpTcpCB);
                
            (4) 解绑 m_iConnSocketFd 与 m_hAsyncIocpQueue(IO 异步队列)
                  HPR_AsyncIO_UnBindIOHandleEx((HPR_HANDLE)m_iConnSocketFd, m_hAsyncIocpQueue);
                  
            (5) 
                /**
                 * HPR_AsyncIO_Send send data by io handle use tcp.
                 * @param IOHandle io handle 
                 * @param pBuffer data pointer
                 * @param BytesToSend data length want to send
                 * @param pUsrData user defined data
                 * pCallBack 函数与 pUsrData 一一对应
                 * @return 0 success, -1 fail
                 * @sa HPR_AsyncIO_Recv()
                 */
                HPR_DECLARE HPR_INT32 CALLBACK HPR_AsyncIO_SendEx(HPR_HANDLE hIOFd, HPR_VOIDPTR pBuffer, 
                                                                  HPR_ULONG BytesToSend, HPR_VOIDPTR pUsrData,
                                                                  HPR_VOIDPTR pCallBack = NULL)
                                                                  
            (6)
                /**
                 * HPR_AsyncIO_Recv recv data by io handle use tcp
                 * @param IOHandle io handle
                 * @param pBuffer data pointer
                 * @param BufferLen data length want to recv
                 * @param pUsrData user defined data
                 * @return 0 success,-1 fail.
                 * @sa HPR_AsyncIO_Send()
                 */
                HPR_DECLARE HPR_INT32 CALLBACK HPR_AsyncIO_RecvEx(HPR_HANDLE hIOFd, HPR_VOIDPTR pBuffer, 
                                                                  HPR_ULONG nBufferLen, HPR_VOIDPTR pUsrData,
                                                                  HPR_VOIDPTR pCallBack = NULL);
                                                                  
                注意: 传入参数中 pUsrData 和 pCallBack 是一一配对的, pUsrData 是作为 pCallBack 函数的参数
                每次调用　HPR_AsyncIO_RecvEx()后，　当对应的 hIOFd socket 的接收到消息, 则 pBuffer　会被赋值, 
                里面的内容是接受的消息, 同时调用 pCallBack 回调函数(pUsrData 作为回调函数的参数).如果想要再次接收数据
                则再次调用 HPR_AsyncIO_RecvEx() 函数
                
             
                
    4. drv_agent_server/StreamMgr/StreamMgr.cpp 里面网络库使用,包含对端地址, port 等
    5. HPR_Atomic.h
                // 原子赋值.
            (1) HPR_DECLARE HPR_UINT32 CALLBACK HPR_AtomicSet(HPR_ATOMIC_T* pDst,HPR_ATOMIC_T nVal);
            
    6. hprplus/StringTraits.h
            主要用于 字符串操作
                  
```

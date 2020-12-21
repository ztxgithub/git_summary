# c++ jsoncpp

## 使用

```shell
(1) json 的构造
     uitdData = mqMsg.toStyledString();
     
     jsonRsp["data"] = Json::Value::null;
     
      构造数组 json
       "{"devCfg": [{"devId":"xxx", "devType":"xxx"}],}"
       
     Json::Value jsonRoot;
     Json::Value jsonDevInfos;
     auto devHBMgrIter = devHBMgr_.begin();
     for (; devHBMgrIter != devHBMgr_.end(); devHBMgrIter++)
     {
     Json::Value devInfo;
     devInfo["devId"] = devHBMgrIter->first;
     devInfo["devType"] = devHBMgrIter->second.hbCfg.devType;
     jsonDevInfos.append(devInfo);
     }
     jsonRoot["devCfg"] = jsonDevInfos;
     content = jsonRoot.toStyledString();
     
     (2) json 的解析 SendMsgToUitd() 函数
     Json::Reader reader;
     Json::Value root;
     reader.parse(strMsg, root)
      文件解析
     is.open(path.c_str(), std::ios::binary);
     if (!reader.parse(is, root)) break;
     if (root["HeartbeatInfos"].isNull()) break;
     
     Json::Value val_array = root["HeartbeatInfos"];
     int iSize = val_array.size();
     string devType;
     for (int nIndex = 0; nIndex < iSize; ++nIndex)
     {
     if (!val_array[nIndex]["devType"].isNull())
     {
     heartBeatCfg hbCfg;
     devType = val_array[nIndex]["devType"].asString();
     hbCfg.heartTimout = val_array[nIndex]["heartTimout"].asInt();
     hbCfg.polltime = val_array[nIndex]["polltime"].asInt();
     devHeartCfg.insert(make_pair(devType, hbCfg));
     }
     }
     (3) json 中是否存在某个字段
     if(val_array[nIndex].isMember("channelNo"))
                {
                    evenInfo.channelNo = val_array[nIndex]["channelNo"].asString();
                }
     object.isMember("channelNo")
```

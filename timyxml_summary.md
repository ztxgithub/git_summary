# timyxml 使用
```shell
    1. 
        遍历使用
         for( TiXmlNode* pItemInfo = pCapCfgElement->FirstChild("cap"); pItemInfo; pItemInfo = pItemInfo->NextSibling("cap"))
         {
         string id = GetAttrFromNode(pItemInfo, "id");
         set<int> setEvent;
         for( TiXmlNode* pItem = pItemInfo->FirstChild("event"); pItem; pItem = pItem->NextSibling("event"))
         {
         string idx = GetAttrFromNode(pItem, "idx");
         setEvent.insert(atoi(idx.c_str()));
         m_mapCapAndEvents[id] = setEvent;
         DAG_DEBUG("-id = %s , idx = %s",id.c_str(),idx.c_str());
         }
         }
         
         
            TiXmlElement* pElementNoReconnectDevList = pRootElement->FirstChildElement("NoReconnectDevList");
                if (pElementNoReconnectDevList != NULL)
                {
                    for (TiXmlElement* pElementDevType = pElementNoReconnectDevList->FirstChildElement("DevType"); pElementDevType != NULL; pElementDevType = pElementDevType->NextSiblingElement())
                    {
                        if (pElementDevType->GetText())
                        {
                            std::string strDevType = pElementDevType->GetText();
                            DAG_DEBUG("- %s", strDevType.c_str());
                            m_noconnectDevtypeList.insert(strDevType);
                        }
                    }
                }
                
        (2)
        
            <Configuration>
            <search>
                <expireTime>0</expireTime>
                <interval>30</interval>
                <deviceNum>500</deviceNum>
            </search>
            </Configuration>
            
            
            if (!oXml.LoadFile())
            {
                DAF_ERROR("[0x%08x] - [path=%s,errorId=%d,errorDesc=%s]load config file fail",
                    ERROR_CODE_OPEN_FILE_FAIL, PROTOCOL_COFIG, oXml.ErrorId(),oXml.ErrorDesc());
                break;
            }
    
            const TiXmlElement* oRoot = oXml.RootElement();//Configuration
             
            const TiXmlElement* pSearchInfo = oRoot->FirstChildElement("search");
            if (pSearchInfo != NULL)
            {
                const TiXmlElement* pNode = pSearchInfo->FirstChildElement();
                while(pNode != NULL)
                {
                    string strName = pNode->ValueStr();
                    string strText = pNode->GetText();
                    if (!strName.empty() && !strText.empty() )
                    {
                        m_mapConfig[strName] = strText;
                    }
                    pNode = pNode->NextSiblingElement();
                }
            }
            
           
        (3)
            pProtos = oRoot->FirstChildElement(CFG_PROTOCOLS);
            pProtos = pProtos->FirstChildElement();//protocol
            while(NULL != pProtos)
            {
                string strTmp = pProtos->Attribute(CFG_PROTO_ENALBE);
                if (strTmp != "true")
                {
                    pProtos = pProtos->NextSiblingElement();  // 
                    continue;
                }
    
                const char* pszTmp = pProtos->Attribute(CFG_PROTO_ID);
                if (pszTmp != NULL)
                {
                    m_mapConfig[CFG_PROTO_ID] = pszTmp;
                }
    
                const TiXmlElement* pNode = pProtos->FirstChildElement();
                while(pNode != NULL)
                {
                    string strName = pNode->ValueStr();
                    string strText = pNode->GetText();
                    if (!strName.empty() && !strText.empty() )
                    {
                        m_mapConfig[strName] = strText;
                    }
                    pNode = pNode->NextSiblingElement();
                }
                
                return true;//目前只支持一种协议
    
            }
```

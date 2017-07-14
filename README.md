# juniper srx virtual-router configure
routing-instances {
    cernet {                               //1.VR的名称，你们之前用的是 vr-untrust-sdsl
        instance-type virtual-router;      //2. 尽量用virtual-router
        interface ge-0/0/10.0;             //3.入接口，ADSL所在接口
        routing-options {
            interface-routes {
                rib-group inet route-share; //4.名字随便写，要跟后面的routing-option合并时一
            }
            static {
                route 0.0.0.0/0 {
                    next-hop 202.119.26.253; //5.地址为ADSL IP地址
                    qualified-next-hop 222.190.112.97 {  //6.VIP line 地址，ADSLdown掉后自动切换回VIP line
                        preference 50;
                    }
                }
               
            }
        }
    }

firewall {
    family inet {
        filter policy_route {    // 7.名字随便写，PC所在接口上调用的得是这个FW
           term 1 {                    
                from {
                    destination-address {
                       x.x.x.x/x;                  //8
                    }
                    source-prefix-list {
                        x.x.x.x/x;                 //与8一致
                    }
                }
                then {
                   accept;
                }
            term 2{                   
                from {
                    destination-address {
                        0.0.0.0/0;
                    }
                    source-prefix-list {
                        host;                   //PC 的IP     
                    }
                }
                then {
                    routing-instance cernet;     //参考1
                }
            }
            term default {
              then {
                   accept;

                     }
     }
 
 
ge-0/0/2 {                             //PC 所在接口
    unit 0 {
        family inet {
                filter {
                  input policy_route;  //参考7
            }
        }
    }
}





routing-options {
    interface-routes {
        rib-group inet route-share;       //参考4                      
    }
    rib-groups {
        route-share {
            import-rib [ inet.0 cernet.inet.0  ];  //参考1

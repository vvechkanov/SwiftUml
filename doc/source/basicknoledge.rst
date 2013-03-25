Proxy Request
=============

.. uml::
    actor User
    participant "swift-client"
    participant "Application:Proxy" as proxy
    participant "ObjectController:Proxy" as ObjC
    participant "ContainerController:Proxy"
    participant "AccountController:Proxy"
    "User" -> "swift-client"
    "swift-client" -> "proxy": Request(type? = PUT)
    alt Path is account/container/object
        "proxy" -> "ObjC": PUT(request)
        "ObjC" -> "Ring": get container_info
            "Ring" -> "Ring": get account_info and save in container_info
            "Ring" --> "ObjC": return container_info
        "ObjC" -> "ObjC": calculate partition
        loop "replica_number times"
            alt "There are primary nodes"
                "ObjC" -> "ObjC": node = next local nodes
            else "otherwise"
                "ObjC" -> "ObjC": node = next handoff nodes
            end
            "ObjC" ->> "node": PUT Request
            note over "ObjC": Somewhere here making \nPut to account and container
            note over "ObjC": Somewhere here making \nresponce to user
        end
    else Path is account/container
        "proxy" -> "ContainerController:Proxy":PUT(request)
        "ContainerController:Proxy" -> "Ring": account_info
        "Ring" --> "ContainerController:Proxy": return account_info
    else Path is account
        "proxy" -> "AccountController:Proxy":PUT(request)
    end

Then request comes to servers on different nodes.


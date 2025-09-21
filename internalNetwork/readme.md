# Info
- It was requested for us to create an internal network
- With this in mind we created 2 new components: a new chaincode and a rest component to call this chaincode
- The new chaincode code can be found [Here](https://github.com/pedromnchunks35/form-chaincode)
- The rest component code can be found [Here](https://github.com/pedromnchunks35/hospital-rest)
- Also during this time we have been testing and improving the multi-client component since some bugs arrived there (therefore we must fix them and check the script)

## Form chaincode Methods
- CreateAsset
  - It exists for creating assets in the ledger
    - The data structure is the following:
      ```
       type PostAssetRequest struct {
            Id            string    `json:"id"`
            TypeForm      string    `json:"type_form"`
            Description   string    `json:"description"`
            Timestamp     time.Time `json:"timestamp"`
            InsertionType string    `json:"insertion_type"`
            Hash          string    `json:"hash"`
      }
      ```
- DeleteAssetById
  - It exists for deleting an asset using its id
  - The data structure for this is an `string id`
- GetAllAssets
  - This is a method for getting all the assets in the ledger
  - We can use filters and pagination in it
    - Filter data structure:  
      ```
      type Filter struct {
        Ids            []string        `json:"ids"`
        TypeForms      []string        `json:"type_forms"`
        InsertionTypes []string        `json:"insertion_types"`
        Hashs          []string        `json:"hashs"`
        TimeFilter     TimestampFilter `json:"time_filter"`
      }
      ```
     - The `TimeFilter` structure is the following:
        ```
        type TimestampFilter struct {
          Min time.Time `json:"min"`
          Max time.Time `json:"max"`
        } 
        ```
- GetAssetById
  - This is a method for getting an asset using its id
  - The data structure for this is an `string id` as param
  - The answer has the following structure:
    ```
    type AssetRequest struct {
	  Id            string    `json:"id"`
	  TypeForm      string    `json:"type_form"`
	  Description   string    `json:"description"`
	  Timestamp     time.Time `json:"timestamp"`
	  InsertionType string    `json:"insertion_type"`
	  Hash          string    `json:"hash"`
    } 
    ```
- GetHistoryAssetById
  - This is a method for getting an asset history so far
  - The data structure for the response is the following (note that it will be in array form):
    ```
    type KeyModification struct {
	  TxId                 string                 `protobuf:"bytes,1,opt,name=tx_id,json=txId,proto3" json:"tx_id,omitempty"`
	  Value                []byte                 `protobuf:"bytes,2,opt,name=value,proto3" json:"value,omitempty"`
	  Timestamp            *timestamppb.Timestamp `protobuf:"bytes,3,opt,name=timestamp,proto3" json:"timestamp,omitempty"`
	  IsDelete             bool                   `protobuf:"varint,4,opt,name=is_delete,json=isDelete,proto3" json:"is_delete,omitempty"`
    }
    ```
- PatchAsset
  - This is a method for changing parameters of an asset individually
  - The request is the following:
    ```
    type PutAssetRequest struct {
     TypeForm      string    `json:"type_form"`
     Description   string    `json:"description"`
     Timestamp     time.Time `json:"timestamp"`
     InsertionType string    `json:"insertion_type"`
     Hash          string    `json:"hash"`
     }
    ```
  - PS: The name PUT does not mean it is a PUT.. but rather a patch (this was a lapse)

## Rest API for calling the form chaincode 
- Note that this is a service that is currently in nodePort since the metalLB does not work correctly within the hospital network for me (this is something that depends a lot on the network configuration)
- For it to work with a statically defined ip we may need to set metalLB in BGP mode [more info here](https://metallb.universe.tf/configuration/_advanced_bgp_configuration/)
  - This is because the L2 mode is not working (maybe the internal network denies ARP or NDP requests for external ips)
- Either way in node port mode we can still access the service from the external network but for that to happen we must know which node has the service running (this can be found by `kubectl describe` command)
  - We can technically also obligate the network to put this service in the main node and that's it (which is cool but maybe not the best) 
  - Currently, the service is working in `chp-srvaida70/172.21.220.46`
  - To access it we must use the ip with port :30030
  - Which means that the url is http://chp-srvaida70:30030
### Routes
#### GET requests
- "/:page/:size"
  - This is for getting all the assets
  - You should pass a filter
  - The following structure is the filter
  ```
      type Filter struct {
	    Ids            []string        `json:"ids"`
	    TypeForms      []string        `json:"type_forms"`
	    InsertionTypes []string        `json:"insertion_types"`
	    Hashs          []string        `json:"hashs"`
	    TimeFilter     TimestampFilter `json:"time_filter"`
      }
  ```
- "/getById/:id"
  - This is for getting the asset by id
- "/getAssetHistoryById/:id"
  - This is for getting the history of the asset by id

#### POST requests
- ""
  - This is for creating an asset
  - The following structure is what must be passed
    ``` 
    type PostAssetRequest struct {
     Id            string    `json:"id"`
     TypeForm      string    `json:"type_form"`
     Description   string    `json:"description"`
     InsertionType string    `json:"insertion_type"`
     Hash          string    `json:"hash"`
     Timestamp     time.Time `json:"timestamp"`
    }
    ```

#### PATCH requests
- "/patchById/:id"
  - This is for changing individual properties of a given asset
  - The following is the structure that your json should take
    ``` 
    type PostAssetRequest struct {
     Id            string    `json:"id"`
     TypeForm      string    `json:"type_form"`
     Description   string    `json:"description"`
     InsertionType string    `json:"insertion_type"`
     Hash          string    `json:"hash"`
     Timestamp     time.Time `json:"timestamp"`
    }
    ```
    
#### DELETE requests
- "/deleteById/:id"
  - This is for deleting a given asset


# More
- We should either create a script to delete logs from the containers or increase the partition in /var
- This is because once the logs fill up there will be a disk pressure trait and everything will go cabum in the network..

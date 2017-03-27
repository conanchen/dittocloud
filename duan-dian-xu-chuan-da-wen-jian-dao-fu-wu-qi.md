# Upload File Client API

## service Auth \(dittoserver/authenticate/service.proto\)

| 处理登录验证: 相当于oauth v2方法，/v2/oauth?grant\_type=password&client\_id=chitsmarttbox&cient\_secret=dkjfV\_23T&username=234234993&password=123456aA |
| :--- |


| Method | Request Type | Response Type | Desc |
| :--- | :--- | :--- | :--- |
| Authenticate | AuthenticateRequest | AuthenticateResponse | 处理登录验证: |

Message AuthenticateRequest

| Field | Description | Type |
| :--- | :--- | :--- |
| client\_id | 如百度api需要的apikey | String |
| client\_secret | 如百度api需要的apikeysecret | String |
| username | 如tbox终端编号：234234993 | String |
| password | 如tbox终端密码：123456aA | String |

Message AuthenticateResponse

| Fileld | Description | Type |
| :--- | :--- | :--- |
| access\_token | e.g. "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIxNDQyNm...wcXrS5yGtUoewAKqoqL5JhIQ109s1FMNopL\_50HR\_t4" | String in JWT format with claims:       sub: user\_uuid,, client: client\_id |
| expires\_in | e.g. 239200 | long |
| tokey\_type | e.g. "Bearer" | String |

```js
service DittoAuthService {
    //Authenticate the client's user, returns FAIL 
    //if the client's users exceed MAX_USERS or password not matched
    (1) rpc Authenticate(AuthenticateRequest) returns AuthenticateResponse{
    }

    message AuthenticateRequest{
        string client_id = 1;
        string client_secret = 2;
        string username = 3;
        string password = 4;
    }

    message AuthenticateResponse {
        string access_token = 1;
        int64 expires_in = 2;
        string refresh_token =3;
        string token_type = 4;       
    }

    其中sccess_token是通过JWT方式编码，其中主要内容Claims由以下组成
    {
        "iss" : "http://www.dddd.com",
        "exp" : 12341234234,
        "sub" : "kjasdfijijdsij$akV_asdkalsasdvidjfig", (即user_uuid)
        "client" : "smartiosapp", (即client_id)
    }
}
```

## service Upload \(dittoserver/fileupload/service.proto\)

| RPC Method | Request Type | Response Type | Desc |
| :--- | :--- | :--- | :--- |
| CreateFile | CreateFileRequest | DittoFile | 在服务端创建文件 |
| GetFile | GetFileRequest | DittoFile | 获取文件信息 |
| ListFileBlocks | ListFileBlocksRequest | ListFileBlocksResponse | 获取文件块状态信息 |
| UploadFileBlock | UploadFileBlockRequest | DittoFileBlock | 上传文件块 |

dittofile.proto

```js
syntax= "proto3"
package dittocloud.dittofile.v1

option java_multiple_files = true;
option java_outer_classname = "DittoFileProto";
option java_package = "com.dittocloud.dittofile.v1";

service DittoFileService {
    // Creates a file on server, and return the new file
    (1) rpc CreateFile(CreateFileRequest) returns DittoFile { 
    }

    // Gets a file summary, return NOT_FOUND if the file does not exist
    rpc GetFile(GetFileRequest) returns DittoFile {
    }

    // Lists files, 
    rpc ListFiles(ListFilesRequest) returns ListFilesResponse {
    }

    // Lists file blocks, the order sorted by file block no
    rpc ListFileBlocks(ListFileBlocksRequest) returns ListFileBlocksResponse {
    }

    // Deletes a file. Returns NOT_FOUND if the file does not exist
    rpc DeleteFile(DeleteFileRequest) returns (google.protobuf.Empty){
    }

    //upload a file block
    (1) rpc UploadFileBlock(UploadFileBlockRequest) returns DittoFileBlock{
    }

}
```

```js
message DittoFile {
    enum Status { CREATED = 1; UPLOADING = 2; UPLOADED = 3; WRITTEN = 4}

    //The resource uuid of the file
    //The name is ignored when creating a file
    string uuid = 1;
    //The name of the file
    string name = 2;
    //The size of the file
    int32 size = 3;
    //The type of the file
    string type = 4;
    //The MD5 of the file content
    string md5 = 5;
    //status: created,uploading,uploaded
    Status status = 6;
    //The created time of the file
    Timestamp created = 7;
    //The written time of the file
    Timestamp written = 8;
}

message CreateFileRequest{
    //The file to create.
    DittoFile = 1;
}

message GetFileRequest{
    //The uuid of the file to retrieve
    string uuid = 1;
}
```

```js
service DittoFileService {
    message ListFilesRequest{        
        string page_token = 1;

        // Requested page size. Server may return fewer file blocks than requested.
        // If unspecified, server will pick an appropriate default.
        int32 page_size = 2;

        optional DittoFile.Status status = 3;
    }
    message ListFilesResponse{
        // The list of files
        repeated DittoFile files = 1;

        // A token to retrieve next page of results.
        // Pass this value in the [ListFilesRequest.page_token]
        // field in the subsequent call to `ListFiles` method to retrieve the next
        // page of results.
        string next_page_token = 2;
    }
}
```

```js
service DittoFileService {
    message ListFileblocksRequest{   
        string file_uuid = 1;     

        int32 block_no = 2;

        // Requested page size. Server may return fewer file blocks than requested.
        // If unspecified, server will pick an appropriate default.
        int32 page_size = 3;

        optional DittoFileBlock.Status status = 4;
    }

    message ListFileblocksResponse{
        // The list of fileblocks
        repeated DittoFileBlock blocks = 1;

        // A token to retrieve next page of results.
        // Pass this value in the [ListFileBlocksRequest.block_no]
        // field in the subsequent call to `ListFileBlocks` method to retrieve the next
        // page of results.
        int32 next_block_no = 2;
    }

    message DittoFileblock{
        enum Status { NONE = 1; UPLOADED = 3; WRITTEN = 4}

        int32 block_no = 2;
        int32 block_size = 3;
        string md5 = 4;
        Status status = 5;
        byte[] block = 6;
    }

    message UploadFileblockRequest{
        // The uuid of the file in which the fileblock is uploaded.
        string file_uuid = 1;
        // The block to upload
        DittoFileblock block = 2;
    }    
}
```

```js
service DittoWatchService {
    //Reference: https://github.com/coreos/etcd/blob/master/etcdserver/etcdserverpb/rpc.proto

    // Watch watches for events happening or that have happened. Both input and output
    // are streams; the input stream is for creating and canceling watchers and the output
    // stream sends events. One watch RPC can watch on multiple key ranges, streaming events
    // for several watches at once. The entire event history can be watched starting from the
    // last compaction revision.
    (1) rpc Watch(stream WatchRequest) returns (stream WatchResponse) {
        option (google.api.http) = {
          post: "/v1alpha/watch"
          body: "*"
      };
    }

    message WatchRequest {
      // request_union is a request to either create a new watcher or cancel an existing watcher.
      oneof request_union {
        WatchCreateRequest create_request = 1;
        WatchCancelRequest cancel_request = 11;
      }
    }


    message WatchCreateRequest {
      string watch_uuid = 1;

      enum WatchType {
        // receive file writtern event.
        FILE_WRITTEN = 2;
        // receive file uploaded event.
        FILE_UPLOADED = 3;
        FILEBLOCK_UPLOADED = 4;
        FILE_TOUPLOAD_COMMANDED = 5;
        FILEBLOCK_TOUPLOAD_COMMANDED = 6;
      }
      // watches let server only sends back the specified events back to the watcher.
      repeated WatchType watches = 99;
    }

    message WatchCancelRequest {
      // watch_id is the watcher id to cancel so that no more events are transmitted.
      string watch_uuid = 1;
    }

    message WatchResponse {
      string watch_uuid = 1;
      // created is set to true if the response is for a create watch request.
      // The client should record the watch_id and expect to receive events for
      // the created watcher from the same stream.
      // All events sent to the created watcher will attach with the same watch_id.
      bool created = 3;
      // canceled is set to true if the response is for a cancel watch request.
      // No further events will be sent to the canceled watcher.
      bool cancelled = 4;

      repeated Event events = 11;
    }
}
```

```js
message Event {
    string event_uuid;

    //Event created time
    TimeStamp created;

    oneof event_union{
        FileWrittenEvent file_written_event = 1;
        FileUploadedEvent file_uploaded_event = 2;
        FileblockUploadedEvent fileblock_uploaded_event =3;
        FileTouploadCommandedEvent file_toupload_commanded_event = 4;
        FileblockTouploadCommandedEvent fileblock_toupload_commanded_event = 5;        
    }
}
message FileWrittenEvent {
    string file_uuid = 1;
    int64 file_size = 2;
    string md5 = 3;
    TimeStamp created =4;
}

message FileUploadedEvent {
    string file_uuid = 1;
    int64 file_size = 2;
    string md5 = 3;
    TimeStamp uploaded = 4;

}

message FileblockUploadedEvent {
    string file_uuid = 1;
    int32 block_no = 2;
    int32 block_size = 3;
    string md5 = 3;
    TimeStamp uploaded = 4;
}
```

```
message FileTouploadCommandedEvent{
    // start time of file to upload
    TimeStamp start = 1;
    // end time of file to upload
    TimeStamp end = 2;
}

message FileblockTouploadCommandedEvent {

    string file_uuid = 1;
    int32  block_no = 2;
    int32 block_size = 3;
}
```




# First Chapter

## Login:

Request: 通过TLS加密通道上传clientkey,clientsecret,userid,userpassword 

Response: accesstoken: {sub:"tbox/89333923", iss:"http://dittocloud.com",exp:"1300819380"} in JWT

Accesstoken can be a JWT, That's wehre the extra benefit of the encoded metadata comes in.

Now, for the big kicker:**statelessness**. While the server will need to generate the JWT, it does not need to store it anywhere as all of the user metadata is encoded right in to the JWT. The server and client could pass the JWT back and forth and never store it. This scales very well.

### [https://stormpath.com/blog/token-auth-for-java](https://stormpath.com/blog/token-auth-for-java)



# How to Create and verify JWTs in Java

[https://stormpath.com/blog/jwt-java-create-verify](https://stormpath.com/blog/jwt-java-create-verify)






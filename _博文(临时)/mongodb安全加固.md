# mongodb安全加固

## 用户/权限 
```
db.createUser(
   {
     user: "pengyi",
     pwd: "Py123",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
   }
)
```
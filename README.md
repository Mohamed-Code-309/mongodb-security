# mongodb-security

1. [Authentication](#auth)
2. [Authorization](#autho)
3. [Backup and Restore](#backup)



## <a name="auth">1. Authentication</a>
 
### How to Apply/Activate Authentication:

1. MongoDB does not come with a default admin user out of the box, so Before you enable authentication, ensure you create an admin user: 

```mongodb
> use admin;
> db.createUser({
  user: "root",
  pwd: "your_root_password",
  roles: [{ role: "root", db: "admin" }]
});
```
2. MongoDB runs in `open access` mode, allowing all users to perform any operationss.
3. in mongodb config file -`mongod.cfg`- the file located in: _C:\Program Files\MongoDB\Server\6.0\bin_, uncomment `security` section and add this line :
```yaml
security:
  authorization: enabled
```
4. restart the MongoDB server to enforce the updated configuration.

5. when authentication is enabled: running any command like `show collections`, you will get :
`MongoServerError: command listCollections requires authentication`   

6. There are 2 different ways of authentication in mongodb: 
   1. during connection: `mogosh -u -p` 
   2. after connection: `db.auth("USERNAME", "PASSWORD")` => here the user must be inside the database he created on it.



## <a name="autho">2. Authorization</a>

- User Roles Apply Only When `Authentication` is active.
- User Role consist of a group of privileges (Resource + Action, ex: insert on products collection)


### Demo 1:

```mongodb
> use test
> db.createUser({user:"dev", pwd: "dev", roles: ["read"]});
> db.auth("dev", "dev")
> db.products.deleteOne({_id:1}); 
```
Result:
`RESULT: MongoServerError: not authorized on test to execute command deleteOne ...etc`

- In case the user deleted the resource, despite only having the read role, make sure authentication is enabled in your instance. 
- Without authentication, MongoDB does not enforce role-based access control, and any user can execute any operations.

### Demo 2:
```mongodb
show dbs
```
will show only database the user was created on it.


### Built in Roles:
Link to a good article demonstrate all available roles in Mongodb:
[Mongodb RBAC](https://www.bmc.com/blogs/mongodb-role-based-access-control/
)

### Update or Extend User Role:
to update or extend a specific user role:

```mongodb
db.updateUser(
  "<username>",
  {
    roles: [
      { role: "<role>", db: "<database>" },
      ...
    ],
    // Optional additional fields
    customData: { ... }
  }
);

```
example :

```mongodb
db.updateUser("dev", { roles: ["readWrite", { db: "cylingo", role: "read" }] });
```

[1] `db.updateUser("dev", ...):`

This specifies the user to be updated. In this case, the user with the username "dev".


[2] `roles: [...]:`

The `roles` field specifies the updated roles assigned to the user.

[3] `"readWrite":`

This grants the `readWrite` role to the user. The readWrite role allows the user to read and modify data (insert, update, delete) in all collections within the current database (db where the command is run).

[4] `{ db: "cylingo", role: "read" }:`

This grants an additional role that is scoped to the `cylingo` database.

### <a name="backup">3. Backup and Restore</a>
Link to a step by step guid on how to restore and backup Mongodb Database:
[Backup and Restore Article](https://gist.github.com/Mohamed-Code-309/5ec2a5030c26fa386988c6ec1391cd1a#how-to-backup-and-restore-mongodb-databases-floppy_disk)

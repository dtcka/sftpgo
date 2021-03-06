# Dynamic user creation or modification

Dynamic user creation or modification is supported via an external program that can be executed just before the user login.
To enable dynamic user modification, you must set the absolute path of your program using the `pre_login_program` key in your configuration file.

The external program can read the following environment variables to get info about the user trying to login:

- `SFTPGO_LOGIND_USER`, it contains the user trying to login serialized as JSON. A JSON serialized user id equal to zero means the user does not exists inside SFTPGo
- `SFTPGO_LOGIND_METHOD`, possible values are: `password`, `publickey` and `keyboard-interactive`

The program must write, on its the standard output:

- an empty string (or no response at all) if the user should not be created/updated
- or the SFTPGo user, JSON serialized, if you want create or update the given user

Actions defined for user's updates will not be executed in this case.

The JSON response can include only the fields to update instead of the full user. For example, if you want to disable the user, you can return a response like this:

```json
{"status": 0}
```

Please note that if you want to create a new user, the pre-login program response must include all the mandatory user fields.

The external program must finish within 60 seconds.

If an error happens while executing your program then login will be denied.

"Dynamic user creation or modification" and "External Authentication" are mutally exclusive, they are quite similar, the difference is that "External Authentication" returns an already authenticated user while using "Dynamic users modification" you simply create or update a user. The authentication will be checked inside SFTPGo.
In other words while using "External Authentication" the external program receives the credentials of the user trying to login (for example the clear text password) and it need to validate them. While using "Dynamic users modification" the pre-login program receives the user stored inside the dataprovider (it includes the hashed password if any) and it can modify it, after the modification SFTPGo will check the credentials of the user trying to login.

Let's see a very basic example. Our sample program will grant access to the existing user `test_user` only in the time range 10:00-18:00. Other users will not be modified since the program will terminate with no output.

```
#!/bin/bash

CURRENT_TIME=`date +%H:%M`
if [[ "$SFTPGO_LOGIND_USER" =~ "\"test_user\"" ]]
then
  if [[ $CURRENT_TIME > "18:00" || $CURRENT_TIME < "10:00" ]]
  then
    echo '{"status":0}'
  else
    echo '{"status":1}'
  fi
fi
```

Please note that this is a demo program and it might not work in all cases. For example, the username should be obtained by parsing the JSON serialized user and not by searching the username inside the JSON as shown here.


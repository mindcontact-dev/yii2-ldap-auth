# Yii2 LDAP Auth
Simple extension to handle auth over LDAP in Yii 2 applications.

**This extension intended for applications that rely *only* on LDAP authentication and does not support access tokens.**

# Installation

```shell script
composer require "MindContact/yii2-ldap-auth:master"
```

# Example of configuration and a use case
Considering [yii2-app-basic](https://github.com/yiisoft/yii2-app-basic): 

### Configure the component in your configuration file and change user identity class
```php
'components' => [
    ...
    'ldapAuth' => [
        'class' => '\MindContact\Yii2LdapAuth\LdapAuth',
        'host' => 'ldap-server',
        'baseDn' => 'dc=shihadeh,dc=intern',
        'searchUserName' => 'cn=admin,dc=shihadeh,dc=intern',
        'searchUserPassword' => 'test1234',

        // optional parameters and their default values
        'ldapVersion' => 3,             // LDAP version
        'protocol' => 'ldap://',       // Protocol to use           
        'followReferrals' => false,     // If connector should follow referrals
        'port' => 389,                  // Port to connect to
        'loginAttribute' => 'cn',      // Identifying user attribute to look up for
        'ldapObjectClass' => 'inetOrgPerson',  // Class of user objects to look up for
        'timeout' => 10,                // Operation timeout, seconds
        'connectTimeout' => 5,          // Connect timeout, seconds
        'roleMappings' => [
            'cn=Admins,ou=Groups,dc=shihadeh,dc=intern' => 'admin',
            'cn=Maintaners,ou=Groups,dc=shihadeh,dc=intern' => 'operator',
        ],
        'isEnabled' => false,
        'demoUser' => [
            'Id' => 'demo.user',
            'Username' => 'Demo User',
            'Email' => 'demo.user@demo.com',
            'Dn' => 'cn=demo_user,dc=shihadeh,dc=intern',
            'Roles' => ['admin']
        ]
    ],
    ...
    
    'user' => [
        'identityClass' => '\MindContact\Yii2LdapAuth\Model\LdapUser',
        'enableSession' => true,
        'enableAutoLogin' => true,
    ],
    ...
]
```
### Update methods in LoginForm class
```php
use MindContact\Yii2LdapAuth\Model\LdapUser;

...

public function validatePassword($attribute, $params)
{
    if (!$this->hasErrors()) {
        $user = LdapUser::findIdentity($this->username);

        if (!$user || !Yii::$app->ldapAuth->authenticate($user->getDn(), $this->password) {
            $this->addError($attribute, 'Incorrect username or password.');
        }
    }
}

...

public function login()
{
    if ($this->validate()) {
        return Yii::$app->user->login(
            LdapUser::findIdentity($this->username),
            $this->rememberMe
                ? 3600*24*30 : 0
        );
    }
    return false;
}
```

### Verify that user belongs to LDAP group
If you need also need to check if user is a member of certain LDAP group, use one more parameter
for the `authenticate` function:
```php
Yii::$app->ldapAuth->authenticate($user->getDn(), $this->password, 'cn=auth-user-group')
```

Now you can login with LDAP credentials to your application.

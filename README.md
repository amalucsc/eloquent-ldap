# eloquent-ldap

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Software License][ico-license]](LICENSE.md)

A Laravel package that first tries to log the user against the internal 
database, if that fails, it tries against the configured LDAP/AD 
server. Optionally it will create a local user record on first
login of an LDAP user, as well as grant that user permissions
to local groups that have matching names of the LDAP groups
that the user is a member of.


## Install

Via Composer

``` bash
$ composer require sroutier/eloquent-ldap
```

## Declare provider

Add this declaration in the provider array of your `./config/app.php` file:

``` php
        Sroutier\EloquentLDAP\Providers\EloquentLDAPServiceProvider::class,
```

## Publish assets

To publish the assets, config file and migration scripts, run this command:

``` bash
$ php artisan vendor:publish --provider="Sroutier\EloquentLDAP\Providers\EloquentLDAPServiceProvider"
```

This will publish a config file and a migration file.

## Migration

The migration script will add a new column `auth_type` to the schema of the 
`users` table, and one column `resync_on_login` to the `groups` table. You 
should already have both tables, but if you do not or if you want to use
different tables for those purposes, the migration to create those 
tables is provided as an example, but commented out. You will 
want to review the migration script and adjust according to 
your scenario.

Once ready, run the migration script with this command:

``` bash
$ php artisan migrate
```

## Configure

The recommended way to configure this package is by defining the following 
variables in you `.env` file and adjusting the values there. For a 
detailed explanation of each setting, refer to the config file 
that you published above.

The configuration that you will need will vary based on the type or server that you wish to authenticate against.
Below are example config section for both options, Lightweight Directory Access Protocol (LDAP) and Microsoft 
Active Directory (MSAD).

### Microsoft Active Directory server.

Below is a section of a ```.env``` config file that shows how to configure your system to access a Microsoft Active 
Directory server:

```
LDAP.ENABLED=true
LDAP.DEBUG=false
LDAP.SERVER_TYPE=MSAD
LDAP.CREATE_ACCOUNTS=true
LDAP.REPLICATE_GROUP_MEMBERSHIP=true
LDAP.RESYNC_ON_LOGIN=true
LDAP.GROUP_MODEL=App\Models\Role
LDAP.LABEL_INTERNAL=internal
LDAP.LABEL_LDAP=ldap
LDAP_ACCOUNT_SUFFIX=@company.com
LDAP_BASE_DN=DC=department,DC=company,DC=com
LDAP_SERVER=ldapsrv01.company.com
LDAP.PORT=389
LDAP_USER_NAME=ldap_reader
LDAP_PASSWORD=PaSsWoRd
LDAP.RETURN_REAL_PRIMARY_GROUP=true
LDAP.SECURED=false
LDAP.SECURED_PORT=636
LDAP.RECURSIVE_GROUPS=true
LDAP.SSO=false
LDAP.USERNAME_FIELD=samaccountname
LDAP.EMAIL_FIELD=userprincipalname
LDAP.FIRST_NAME_FIELD=givenname
LDAP.LAST_NAME_FIELD=sn
LDAP_USER_FILTER=(&(objectcategory=person)(samaccountname=%username))
```

### Lightweight Directory Access Protocol server.

Below is a section of a ```.env``` config file that shows how to configure your system to access a Lightweight 
Directory Access Protocol server:

```
LDAP.ENABLED=true
LDAP.DEBUG=false
LDAP.SERVER_TYPE=LDAP
LDAP.CREATE_ACCOUNTS=true
LDAP.REPLICATE_GROUP_MEMBERSHIP=false
LDAP.RESYNC_ON_LOGIN=false
LDAP.GROUP_MODEL=App\Models\Role
LDAP.LABEL_INTERNAL=internal
LDAP.LABEL_LDAP=ldap
LDAP.ACCOUNT_SUFFIX=
LDAP.BASE_DN=dc=example,dc=com
LDAP.SERVER=ldap.forumsys.com
LDAP.PORT=389
LDAP.USER_NAME=cn=read-only-admin,dc=example,dc=com
LDAP.PASSWORD=password
LDAP.RETURN_REAL_PRIMARY_GROUP=true
LDAP.SECURED=false
LDAP.SECURED_PORT=636
LDAP.RECURSIVE_GROUPS=true
LDAP.SSO=false
LDAP.USERNAME_FIELD=uid
LDAP.EMAIL_FIELD=mail
LDAP.FIRST_NAME_FIELD=
LDAP.LAST_NAME_FIELD=sn
LDAP.USER_FILTER=(&(objectClass=person)(uid=%username))
```

**_NOTE:_** THe configuration options above will allow you to connect and authenticate users using the publicly 
available OpenLDAP test server hosted by 
[Forum Systems](http://www.forumsys.com/en/tutorials/integration-how-to/ldap/online-ldap-test-server/).

### MSAD vs LDAP

A couple of difference in how to configure the system depending on which server type is being used are worth pointing 
out.

* LDAP.SERVER_TYPE: Can be either LDAP or MSAD. Lets the system know how to interact with the authentication server.
* LDAP.REPLICATE_GROUP_MEMBERSHIP: Currently only supported for MSAD servers.
* LDAP.RESYNC_ON_LOGIN: Currently only supported for MSAD servers.
* LDAP.ACCOUNT_SUFFIX: 
    * LDAP: Should remain empty for LDAP servers. 
    * MSAD: Should contain the static part of the users email address.
* LDAP.USER_NAME: 
    * LDAP: Should be the complete DN of the user to bind with.
    * MSAD: Simply the name of the user to bind with.
* LDAP.RETURN_REAL_PRIMARY_GROUP:
    * LDAP: Not used.
    * MSAD: Fix Microsoft AD not following standards may incur extra processing.

## Usage

The `users` table/model must have the following columns/attributes named 
`username`, `first_name`, `last_name` and `email`. The migration 
script provided with this package has an example of how to 
create such a table but it is commented out.
 
The user model must have the `auth-type` attribute added to its `fillable` array
to allow setting the column in the database.

Also your login view and `AuthController` must accept a user name and password.
They can accept other fields if you want, such as email, security token, 
etc... But the first time a new user tries to log in, since he will not
be found in the local database, the package will need the user name to
authenticate against the LDAP server. 

## Example

For a concrete example of this package used in an active project, see 
[sroutier/laravel-5.1-enterprise-starter-kit](https://github.com/sroutier/laravel-5.1-enterprise-starter-kit).
Note that in that project this package is used in combination with 
[Zizaco/entrust](https://github.com/zizaco/entrust) to provide
role based authorization, therefore there is no group model, 
but instead a role model.

## Change log

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email sroutier@gmail.com instead of using the issue tracker.

## Credits

- [Sebastien Routier](https://github.com/sroutier)
- [All Contributors](https://github.com/sroutier/eloquent-ldap/graphs/contributors)

## License

The GNU General Public License Version 3 (GPLv3). Please see [License File](LICENSE.md) for more information.

[ico-version]: https://img.shields.io/badge/packagist-v0.1.2-orange.svg
[ico-license]: https://img.shields.io/badge/licence-GPLv3-brightgreen.svg

[link-packagist]: https://packagist.org/packages/sroutier/eloquent-ldap

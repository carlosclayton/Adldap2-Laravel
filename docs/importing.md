# Importing

Adldap2-Laravel comes with a command that allows you to import users from your LDAP server automatically.

> **Note**: Make sure you're able to connect to your LDAP server and have configured
> the `adldap` auth driver correctly before running the command.

## Running the Command

To import all users from your LDAP connection simply run `php artisan adldap:import`.

> **Note**: The import command will utilize all scopes and sync all attributes you
> have configured in your `config/adldap_auth.php` configuration file.

Example:

```bash
php artisan adldap:import

Found 2 user(s).
```

You will then be asked:

```bash
 Would you like to display the user(s) to be imported / synchronized? (yes/no) [no]:
 > y
```

Confirming the display of users to will show a table of users that will be imported:

```bash
+------------------------------+----------------------+----------------------------------------------+
| Name                         | Account Name         | UPN                                          |
+------------------------------+----------------------+----------------------------------------------+
| John Doe                     | johndoe              | johndoe@email.com                            |
| Jane Doe                     | janedoe              | janedoe@email.com                            |
+------------------------------+----------------------+----------------------------------------------+
```

After it has displayed all users, you will then be asked:

```bash
 Would you like these users to be imported / synchronized? (yes/no) [no]:
 > y
 
  2/2 [============================] 100%
  
Successfully imported / synchronized 2 user(s).
```

### Importing a Single User

To import a single user, insert one of their attributes and Adldap2 will try to locate the user for you:

```bash
php artisan adldap:import jdoe@email.com

Found user 'John Doe'.
```

## Command Options

### Filter

The `--filter` (or `-f`) option allows you to enter in a raw filter in combination with your scopes inside your `config/adldap_auth.php` file:

```bash
php artisan adldap:import --filter "(cn=John Doe)"

Found user 'John Doe'.
```

### No Logging

The `--no-log` option allows you to disable logging during the command.

By default, this is enabled.

```bash
php artisan adldap:import --no-log
```

### Delete

The `--delete` (or `-d`) option allows you to soft-delete deactivated LDAP users. No users will
be deleted if your User model does not have soft-deletes enabled.

```bash
php artisan adldap:import --delete
```

### Restore

The `--restore` (or `-r`) option allows you to restore soft-deleted re-activated LDAP users.

```bash
php artisan adldap:import --restore
```

> **Note**: Usually the `--restore` and `--delete` options are used in tandem to allow full synchronization.

### No Interaction

To run the import command via a schedule, use the `--no-interaction` flag:

```php
php artisan adldap:import --no-interaction
```

Users will be imported automatically with no prompts.

You can also call the command from the Laravel Scheduler, or other commands:

```php
// Importing one user
$schedule->command('adldap:import sbauman', ['--no-interaction'])
            ->everyMinute();
```

```php
// Importing all users
$schedule->command('adldap:import', ['--no-interaction'])
            ->everyMinute();
```

```php
// Importing users with a filter
$dn = 'CN=Accounting,OU=SecurityGroups,DC=Acme,DC=Org';

$filter = sprintf('(memberof:1.2.840.113556.1.4.1941:=%s)', $dn);

$schedule->command('adldap:import', ['--no-interaction', '--filter' => $filter])
    ->everyMinute();
```

## Tips

 - Users who already exist inside your database will be updated with your configured `sync_attributes`
 - Users are never deleted from the import command, you will need to delete users regularly through your model
 - Successfully imported (new) users are reported in your log files with:
  - `[2016-06-29 14:51:51] local.INFO: Imported user johndoe`
 - Unsuccessful imported users are also reported in your log files, with the message of the exception:
  - `[2016-06-29 14:51:51] local.ERROR: Unable to import user janedoe. SQLSTATE[23000]: Integrity constraint violation: 1048`
  - Specifying a username uses ambiguous naming resolution, so you're able to specify attributes other than their username, such as their email (`php artisan adldap:import jdoe@mail.com`).
  - If you have a password mutator (setter) on your User model, it will not override it. This way, you can hash the random 16 characters any way you please.


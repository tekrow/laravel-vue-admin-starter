```
1. Run Migrations and Seeders
```shell
composer install
php artisan key:generate
php artisan migrate
php artisan db:seed --class SeedAdminRoleAndUser
```
2. Now build the npm dependencies using `vitejs`:
```sh
npm i
npm run serve

```

## Usage
### The initial seeded admin user and role
When you run `php artisan vendor:publish --tag=lvg-migrations`, a migration is published that creates an initial default user called `Administrator` and a role with the name `administrator` to enable you gain access to the system with admin privileges. The credentials for the user account are:
* Email: **admin@tekrow.com**
* Password: **password**

Use these creds after migration to login and explore all parts of the application

### Create the Permissions, Roles and Users Modules first, in that order:
Run the following commands to generate the User Access Control Module before proceeding to generate your admin:
```shell
php artisan lvg:generate:permission -f
php artisan lvg:generate:role -f
php artisan lvg:generate:user -f
```
You can now proceed to generate any other CRUD you want using the steps in the following section.
### General Steps to generate a CRUD:
1. Generate and write a migration for your table with `php artisan make:migration` command.
2. Run the migration to the database with `php artisan migrate` command
3. Generate the Whole Admin Scaffold for the module with `php artisan lvg:generate` command
4. Modify and customize the generated code to suit your specific requirements if necessary.
__NB: If the crud already exists, and you would like to generate, you can use the `-f` or `--force` option to force replacement of files.
### Example
Assuming you want to generate a `books` table:
```shell
php artisan make:migration create_books_table
```


* Open your migration and modify it as necessary, adding your fields. After that, run the migration.
```shell
php artisan migrate
```
* __The Fun Part:__ Scaffold a whole admin module for books with lvg using the following command:
```shell
php artisan lvg:generate books #Format: php artisan generate [table_name] [-f]
```
__NB:__ To get a full list of `lvg` commands called under the hood and the full description of the `lvg:generate` command, you can run the following: 
```shell
php artisan lvg --help
php artisan lvg:generate --help
```
The command above will generate a number of files and add routes to both your `api.php` and `web.php` route files. It will also append menu entries to the published `Menus.json` file.
The generated vue files are placed under the Pages/Books folder in the js folder.

* Finally, run `yarn dev or yarn build` to compile the assets. There you have your CRUD.
## Roles, permissions and Sidebar Menu:
* By Default, generation of a module generates the following permissions:
    - index
    - create
    - show
    - edit
    - delete
    
* The naming convention for permissions is ${module-name}.${perm} e.g payments.index, users.create etc.
* This package manages access control using policies. Each generated module generates a policy with the default laravel actions:
    - viewAny, view, store, update, delete, restore, forceDelete
  The permissions generated above are checked in these policies. If you need to modify any of the access permissions, policies is where to look.
      
* Special permissions MUST also be generated to control access to the sidebar menus. These permissions SHOULD NOT contain two parts separated with a dot, but only one part.
* Menus are configured in a json file published at `./resources/js/Layouts/Lvg/Menu.json`. 
    - For all menu items, the json key MUST match the permision that controls that menu. A permission without any verb is generated when generating each module for this very purpose. For example, generating a `payments` module will generate a `payments` permission.
      Then the menu for payments must have `payments` as the json key.
    - For parent menus and any other menus which may not match any module, you have to create a permission with the key name to control its access. For example, if you have a parent menu called `master-data` you have to generate a permission with the same name.

## Components Documentation
### Datatables
LVG is built on top of datatables.net and is fully server-side rendered using [Yajra Datatables](https://yajrabox.com/docs/laravel-datatables/master/introduction). Most of the logic resides inside `App\Repositories` and in the respective Repository file, there is a method called `dtColumns` which is used to fully control the columns shown in the Index page.

For example, in order to control the columns shown for the Users Datatable, the following is the `dtColumns` method under `App\Repositories\Users.php`:
```php
public static function dtColumns(): array
    {
        return [
            Column::make('id')->title('ID')->className('all text-right'),
            Column::make("name")->className('all'),
            Column::make("first_name")->className('none'),
            Column::make("last_name")->className('none'),
            Column::make("middle_name")->className('none'),
            Column::make("username")->className('min-desktop-lg'),
            Column::make("email")->className('min-desktop-lg'),
            Column::make("gender")->className('min-desktop-lg'),
            Column::make("dob")->className('none'),
            //Column::make("email_verified_at")->className('min-desktop-lg'),
            Column::make("activated")->className('min-desktop-lg'),
            Column::make("created_at")->className('min-tv'),
            Column::make("updated_at")->className('min-tv'),
            Column::make('actions')->className('min-desktop text-right')->orderable(false)->searchable(false),
        ];
    }
```
**NOTE: In order to omit the `email_verified_at` class from my Index columns all I had to do is comment it out (or better yet, just remove it from the list of columns!)**

The datatables are also responsive by default (Checkout https://datatables.net/extensions/responsive/ for more details). For this purpose, you can use one of the following lvg-provided responsive breakpoints to automatically collapse the column below a given screen size. For info on how to use the class logic, checkout the [Class Logic Documentation](https://datatables.net/extensions/responsive/classes). Most of the time I only use `min-`, e.g `min-desktop-l`
```ts
breakpoints: [
        { name: "tv", width: Infinity },
        { name: "desktop-l", width: 1536 },
        { name: "desktop", width: 1280 },
        { name: "tablet-l", width: 1024 },
        { name: "tablet-p", width: 768 },
        { name: "mobile-l", width: 480 },
        { name: "mobile-p", width: 320 },
    ],
}
```
Checkout the first snippet on how I have used the responsive classes!


### Testing

``` bash
composer test
```

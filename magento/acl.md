# Magento Acl

## Menú
[Crea una regla de lista de control de acceso (ACL)](#crea-una-regla-de-lista-de-control-de-acceso)
[Crear archivo acl](#crear-archivo-acl)
[Crear un rol por código](#crear-un-rol-por-código)
[Restringir el acceso a usuarios](#restringir-el-acceso-a-usuarios)
[Restringir Controladores](#restringir-controladores)
[Restringiendo contenido](#restringiendo-contenido)
[Zend Acl](#zend-acl)


# Crea una regla de lista de control de acceso

las listas de control de acceso en Magento 2 es utilizan para definir los permisos para usuarios especificos en el area de administración de magento.
Se puede usar el ACL, para controlar qué acciones puede realizar un usuario, tal como ver menús, acceder a controladores, endponds (api) y layout blocks condicionales.

## Crear archivo acl
Se debe definir los recursos por medio del archivo `etc/acl.xml` del modulo. En caso de que no exista este archivo en la carpeta etc de su modulo, se debera de crear.

La etiqueta resorce debera contener los siguientes atributos

| Atributo | Descripcion |
| ---- | ---- |
| id | String unico que debe tener el siguiente formato: **{Vendor}_{Module}::{resource_id}** |
| title | Título que se muestra en la barra de menú |
| sortOrder | Posición en la que se muestra el menú |

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
  <acl>
    <resources>
      <resource id="Magento_Backend::admin">
        <resource id="{Vendor}_{Module}::{resource_id}" title="{Resource Title}" sortOrder="100">
          <resource id="{Vendor}_{Module}::{sub_resource_id}" title="{Sub Resource Title}" sortOrder="10"/>
        </resource>
      </resource>
    </resources>
  </acl>
</config>
```
Recuerde que después de esto, se debe limpiar la cache `bin/magento cache:clear` o haciendo clic en `Sistema > Administración de caché > Vaciar caché de Magento`

### Crear un rol por código

Este paso se ejemplifica el como crear un rol por medio de código por medio de un upgradeData. Recordemos que para este paso se debe
subir la versón del modulo y correr `bin\magento setup:upgrade`
```php
<?php

namespace {Vendor}\{Module}\Setup;

use Magento\Authorization\Model\Acl\Role\Group
use Magento\Authorization\Model\RoleFactory;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;
use Magento\Framework\Setup\UpgradeDataInterface;
use Magento\Authorization\Model\UserContextInterface;

class UpgradeData implements UpgradeDataInterface
{
    /**
     * @var RoleFactory
     */
    private $roleFactory;

    /**
     * @param RoleFactory $roleFactory
     */
    public function __construct(
        RoleFactory $roleFactory
    ) {
        $this->roleFactory = $roleFactory;
    }

    /**
     * En este ejemplo, usamos la instancia de Role en el cual se usan los siguientes metodos para crear el rol
     * setName: Es el nombre del rol que se va a crear.
     * setRoleType: Aquí se define el grupo al que va a pertenecer el rol
     * setUserType: Aquí se define el tipo de usuario al que va a pertenecer el rol
     *
     * El array de resouces son los modulos o recursos a los que el rol podra tener acceso.
     */
    public function upgrade(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();

        /** @var Magento\Authorization\Model\Role $role */
        $role = $this->roleFactory->create();
        $role->setName('{Role Name}')
            ->setRoleType(Group::ROLE_TYPE)
            ->setUserType(UserContextInterface::USER_TYPE_ADMIN)
            ->setPid(0);

        $resources = [
            '{Vendor}_{Module}::{resource_id}',
            '{Vendor}_{Module}::{sub_resource_id}',
        ];

        $role->setResources($resources);
        $role->save();

        $setup->endSetup();
    }
}
```

## Restringir el acceso a usuarios

En le modulo, cree el siguiente archivo `etc/adminhtml/menu.xml`, este archivo creara un menú que estará oculto para los usuarios no autorizados. Los atributos en los nodos determinan a que recursos pueden acceder

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
    <menu>
     <add id="{Vendor}_{Module}::menu" title="Custom Menu" module="{Vendor}_{Module}" sortOrder="10" resource="{Vendor}_{Module}::menu"/>
     <add id="{Vendor}_{Module}::create" title="Create" module="{Vendor}_{Module}" sortOrder="10" parent="{Vendor}_{Module}::menu" action="custommenu/create/index" resource="{Vendor}_{Module}::create"/>
     <add id="{Vendor}_{Module}::delete" title="Delete" module="{Vendor}_{Module}" sortOrder="20" parent="{Vendor}_{Module}::menu" action="custommenu/delete/index" resource="{Vendor}_{Module}::delete"/>
     <add id="{Vendor}_{Module}::view" title="View" module="{Vendor}_{Module}" sortOrder="30" parent="{Vendor}_{Module}::menu" action="custommenu/view/index" resource="{Vendor}_{Module}::view"/>
    </menu>
</config>
```

| Atributo | Descripcion |
| ------ | ------ |
| id | String unico que debe tener el siguiente formato: **{Vendor}_{Module}::{resource_id}** |
| title | Título que se muestra en la barra de menú |
| module | Módulo que contiene el menú actual |
| sortOrder | Posición en la que se muestra el menú |
| Parent| El menú que padre del actual 
| action | la URL de la pagina que debe mostrarce después de hacer click en el menú. Debe tener el siguiente formato: **{front_name}/{controller_path}/{action}** |
| resource | Regla ACL para restringir el acceso **{Vendor}_{Module}::{subresource_id}** |

![menu](https://developer.adobe.com/commerce/php/static/e8a55a7be4c036279716744e897ec5d5/eeba7/custom_menu.webp)


### Restringir Controladores

Para poder aplicar el acceso a un controlador, se puede hacer de dos formas, una de ellas es sobre escribiendo la funcion **_isAllowed** y la otra es sobre escribiendo la constante **ADMIN_RESOURCE**. Esta segunda opción funciona dado a que la funcion _isAllowed esta escrita de la siguiente forma:

```php
    /**
     * Determines whether current user is allowed to access Action
     *
     * @return bool
     */
    protected function _isAllowed()
    {
        return $this->_authorization->isAllowed(static::ADMIN_RESOURCE);
    }
```

```php
<?php

namespace {Vendor}\{Module}\Controller\Adminhtml;

use Magento\Backend\App\Action;
use Magento\Backend\App\Action\Context;
use Magento\Framework\App\ResponseInterface;
use Magento\Framework\View\Result\PageFactory;

class Crud extends Action
{
	/** Por medio de sobre escribir la constante */
	const ADMIN_RESORUCE = '{Vendor}_{Module}::{subresource_id}';
}
```

Y al sabre escribir la funcion, seria de la siguiente forma:

```php
<?php

namespace {Vendor}\{Module}\Controller\Adminhtml;

use Magento\Backend\App\Action;
use Magento\Backend\App\Action\Context;
use Magento\Framework\App\ResponseInterface;
use Magento\Framework\View\Result\PageFactory;

class Crud extends Action
{
	/**
     * @override to allow access to this controller
     * @return bool
    */
	protected function _isAllowed()
   {
   		return $this->_authorization->isAllowed('{Vendor}_{Module}::{subresource_id}');
   }
}
```

### Restringiendo contenido

Con los ACL también es posible restringir el render de blockes dinamicos en una pagina, ya que basta con establecer el valor del subresorce en el tag `aclResource`

```xml
<block class="{Vendor}\{Module}\Block\Adminhtml\Type" name="block.example" aclResource="{Vendor}_{Module}::{subresource_id}">
    <!-- ... -->
</block>
```



# Zend Acl

Si bien, es bieno saber como implementar el acl en magento, también es interesante entender como funciona, ya que la clase [Magento\Framework\Acl](#codigo_1) extiende de Zend_Acl de Zend Framework version 1.2, gracias a ello, Magento es capaz de proporcionar una implementación liviana y flexible para la administración de privilegios. En general, una aplicación puede utilizar tales ACL para controlar el acceso a ciertos objetos protegidos por parte de otros objetos solicitantes.

Para poner un poco de contexto, definamos lo siguiente:

- un **Rol** es un objeto que puede solicitar acceso a un recurso.
- un **Recurso** es un objeto al que se le controla el acceso.

Bajo este contexto, los roles solicitan acceso a los recursos, 

#### codigo 1
```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
namespace Magento\Framework;

/**
 * ACL. Can be queried for relations between roles and resources.
 *
 * @api
 * @since 100.0.2
 */
class Acl extends \Zend_Acl
{

```

Para poner un ejemplo, podríamos suponer que para un archivo Foo, que en este caso, sería el recurso, el usuario A tiene todos los privilegios. El usuario B tiene acceso de solo lectura, mientras que el usuario C no tiene acceso al archivo.

En este ejemplo, la ACL define las políticas de control de acceso para el archivo y determina a qué usuarios se les concede o deniega el acceso al archivo y qué acciones pueden realizar (como leer, escribir o ejecutar el archivo).

En zend, podríamos ejemplificar este ejemplo de la siguiente forma:

```php
$acl = new Zend_Acl();
$acl->addRole(new Zend_Acl_Role('A'));
$acl->addRole(new Zend_Acl_Role('B'));
$acl->addRole(new Zend_Acl_Role('C'));

$acl->addResource(new Zend_Acl_Resource('Foo'));
$acl->allow('A', 'Foo', ['read', 'write', 'delete']);
$acl->allow('B', 'Foo', ['read']);
$acl->deny('C', 'Foo');

$acl->isAllowed('A', 'Foo', 'read'); // true
$acl->isAllowed('A', 'Foo', 'write'); // true
$acl->isAllowed('A', 'Foo', 'delete'); // true
$acl->isAllowed('B', 'Foo', 'read'); // true
$acl->isAllowed('B', 'Foo', 'write'); // false
$acl->isAllowed('B', 'Foo', 'delete'); // false
$acl->isAllowed('C', 'Foo', 'read'); // false
$acl->isAllowed('C', 'Foo', 'write'); // false
$acl->isAllowed('C', 'Foo', 'delete'); // false
```

<div class='article-menu'>
  <ul>
    <li>
      <a href="#overview">Almacenamiento de datos en la sesión</a> 
      <ul>
        <li>
          <a href="#start">Iniciando sesión</a>
          <ul>
            <li>
              <a href="#start-factory">Factory</a>
            </li>
          </ul>
        </li>
        <li>
          <a href="#store">Almacenamiento/recuperación de datos en sesión</a>
        </li>
        <li>
          <a href="#remove-destroy">Destrucción/eliminación de sesiones</a>
        </li>
        <li>
          <a href="#data-isolation">Aislamiento de datos de la sesión entre aplicaciones</a>
        </li>
        <li>
          <a href="#bags">Bolsas de sesión</a>
        </li>
        <li>
          <a href="#data-persistency">Datos persistentes en componentes</a>
        </li>
        <li>
          <a href="#custom-adapters">Implementar tus propios adaptadores</a>
        </li>
      </ul>
    </li>
  </ul>
</div>

<a name='overview'></a>

# Almacenamiento de datos en la sesión

El componente de sesión proporciona contenedores orientados a objetos para acceder a datos de la sesión.

Las razones para utilizar este componente en lugar de sesiones "crudas":

* Aislar fácilmente datos de sesión en las aplicaciones en el mismo dominio
* Interceptar donde se establecen los datos de la sesión en su aplicación
* Cambiar el adaptador de la sesión según las necesidades de aplicación

<a name='start'></a>

## Iniciando sesión

Algunas aplicaciones requieren sesiones intensivas, casi cualquier acción que realiza requiere acceso a datos de la sesión. Hay otras que acceden a datos de la sesión casualmente. Gracias al contenedor de servicio, podemos asegurar que la sesión se accede sólo cuando sea claramente necesario:

```php
<?php

use Phalcon\Session\Adapter\Files as Session;

// Iniciar sesión por primera vez cuando algún componente solicite el servicio de session
$di->setShared(
    'session',
    function () {
        $session = new Session();

        $session->start();

        return $session;
    }
);
```

<a name='start-factory'></a>

## Factory

Carga la clase de adaptador de sesión utilizando la opción `adapter`

```php
<?php

use Phalcon\Session\Factory;

$options = [
    'uniqueId'   => 'my-private-app',
    'host'       => '127.0.0.1',
    'port'       => 11211,
    'persistent' => true,
    'lifetime'   => 3600,
    'prefix'     => 'my_',
    'adapter'    => 'memcache',
];

$session = Factory::load($options);
```

<a name='store'></a>

## Almacenamiento/recuperación de datos en sesión

Desde un controlador, una vista o cualquier otro componente que se extienda de `Phalcon\Di\Injectable` se puede acceder al servicio de sesión y almacenar elementos y recuperarlos, como vemos en el siguiente ejemplo:

```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function indexAction()
    {
        // Definir una variable en sesión
        $this->session->set('user-name', 'Michael');
    }

    public function welcomeAction()
    {
        // Comprobar si una variable esta definida
        if ($this->session->has('user-name')) {
            // Obtener dicho valor
            $name = $this->session->get('user-name');
        }
    }

}
```

<a name='remove-destroy'></a>

## Destrucción/eliminación de sesiones

También es posible quitar variables específicas o destruir toda la sesión:

```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function removeAction()
    {
        // Eliminar una variable
        $this->session->remove('user-name');
    }

    public function logoutAction()
    {
        // Destruir toda la sesión
        $this->session->destroy();
    }
}
```

<a name='data-isolation'></a>

## Aislamiento de datos de la sesión entre aplicaciones

A veces un usuario puede utilizar la misma aplicación dos veces en el mismo servidor, en la misma sesión. Seguramente, si utilizamos variables de sesión, queremos que cada aplicación tenga sus datos de sesión separados (aunque el mismo código y mismos nombres de variable). Para solucionar esto, puede añadir un prefijo para cada variable de sesión creada en una aplicación determinada:

```php
<?php

use Phalcon\Session\Adapter\Files as Session;

// Aislamos los datos de la sesión
$di->set(
    'session',
    function () {
        // Todas las variables de la sesión tendrán el prefijo 'my-app-1'
        $session = new Session(
            [
                'uniqueId' => 'my-app-1',
            ]
        );

        $session->start();

        return $session;
    }
);
```

No es necesario agregar un ID único.

<a name='bags'></a>

## Bolsas de sesión

`Phalcon\Session\Bag` es un componente que ayuda a separar datos de una sesión en `namespaces`. Trabajando de esta forma que usted puede crear fácilmente grupos de variables de sesión en la aplicación. Sólo estableciendo las variables en la `bag`, se almacena automáticamente en sesión:

```php
<?php

use Phalcon\Session\Bag as SessionBag;

$user = new SessionBag('user');

$user->setDI($di);

$user->name = 'Kimbra Johnson';
$user->age  = 22;
```

<a name='data-persistency'></a>

## Datos persistentes en componentes

Los controladores, componentes y clases que se extienden de `Phalcon\Di\Injectable` pueden inyectar un `Phalcon\Session\Bag`. Esta clase aísla variables para cada clase. Gracias a esto pueden persistir datos entre peticiones en cada clase de forma independiente.

```php
<?php

use Phalcon\Mvc\Controller;

class UserController extends Controller
{
    public function indexAction()
    {
        // Crear una variable persistente 'name'
        $this->persistent->name = 'Laura';
    }

    public function welcomeAction()
    {
        if (isset($this->persistent->name)) {
            echo 'Bienvenido, ', $this->persistent->name;
        }
    }
}
```

En un componente:

```php
<?php

use Phalcon\Mvc\User\Component;

class Security extends Component
{
    public function auth()
    {
        // Crear una variabl persistente 'name'
        $this->persistent->name = 'Laura';
    }

    public function getAuthName()
    {
        return $this->persistent->name;
    }
}
```

Los datos agregados en la sesión (`$this->session`) están disponibles a través de la aplicación, mientras que el componente persistente (`$this->persistent`) sólo se puede acceder en el ámbito de la clase actual.

<a name='custom-adapters'></a>

## Implementar tus propios adaptadores

Debe implementar la interfaz `Phalcon\Session\AdapterInterface` para crear sus propios adaptadores de sesión o extender los ya existentes.

Hay disponibles más adaptadores para estos componentes en la [Incubadora de Phalcon](https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Session/Adapter)
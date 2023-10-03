
# Integracion Impruvex Provaltec y Manager

Impruvex a llegado a mantener un control exacto y llevar la logistica a otro nivel en Provaltec. Pero tambien llego a sacudir el area de IT con una integracion de ambos sistemas Legacy (cotizador,comex) a Impruvex y Legacy(cotizador,comex) a Manager. En esta documentacion hablaremos y detallaremos toda la integracion, estara dividida en 3 partes, primero el consumo de las APIs de Impruvex, la integracion con Manager y como ultimo la creacion de nuestra API.
 
 

### Temario
- Obtencion de token
- Creacion de OC
- Creacion de OS
- Resumen de inventario
- Envios a nuestra Api
- Creacion OC en Manager
- Creacion de Nota de venta en Manager


### Lenguajes usados

**Legacy:** PHP, Mysql, Bootstrap

**Api:** Node JS

![Logo](https://provaltec.samurai.cl/img/logo/logo-header-2021-03-24-14-26-05-le76317144-2.jpg)
![Logo](https://demoinvas.impruvex.com/plataforma/images/newlogo.png)
![Logo](https://www.manager.cl/wp-content/uploads/2023/07/5-1.webp)




## Consumo de Apis Impruvex

Impruvex nos ha facilitado sus Apis para inyectar la informacion que se ingrese en nuestro sistema, Pero necitamos una token de acceso el cual se entrega mediante el inicio de sesion. Es por eso que se necesita unas credenciales de acceso. Estas credenciales fueron entregadas en la capacitacion realizada por el Maestro Marcelo Fierro y es un usuario especialmente para la integracion. A continuacion detallo como se solicita el token.


### Obtener token

```http
  POST /api/invas/rest/usuario/loginWS
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|

#### Codigo PHP usando CURL

```PHP
$inserData = [
    "usuario" => "tuUsuario",
    "password" => "tuClave"
];

$conInv = curl_init($url);
$headers = [];
$headers[] = 'Content-Type:application/json';
curl_setopt($conInv, CURLOPT_HTTPHEADER, $headers);
curl_setopt($conInv, CURLOPT_POSTFIELDS, json_encode($inserData));
curl_setopt($conInv, CURLOPT_RETURNTRANSFER, true);
$data = curl_exec($conInv);
$result = json_decode($data);
$r = $result->token;
curl_close($conInv);
```
#### Guardar el token 

| Tipo Variable | Codigo     |
| :-------- | :------- | 
| `Session` | $_SESSION["token"] = $result->token | 
| `LocalStorage` | echo ("<script>localStorage.setItem('token','$r');</script>"); | 
| `Variable` | $r = $result->token; |


### Creacion de OC

Solo indicaremos los compos que son requeridos, para mayor detalle solicitar la coleccion a Impruvex

```http
  POST /api/invas/rest/oc/upload
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|
| `codigo` | `string` | Hace referencia al numero de PO en el sistema Comex|
| `sitio` | `string` | El sitio siempre sera CD_PROVALTEC|
| `proveedor` | `string` |  El rut del proveedor, en Comex no tenemos rut, por lo que hay que agregar un campo a la tabla|
| `tipo` | `string` | El tipo es IMPORTADO,pero estamo evaluando la opcion de pasar lo nacional tambien por el modulo de Comex  |
| `descripcion` | `string` | Descripcion de la compra|
| `sku` | `string` | Id del producto|
| `unidadesEnviadas` | `string` | Cantidades que se compraron|
| `campo01` | `string` | No se usa, pero es requerido|
| `campo02` | `string` | No se usa, pero es requerido |

#### Consulta MYSQL

```mysql
//  Cabecera 
$query = "SELECT CO.ocompra,CP.id FROM comex_ocompra CO JOIN comex_proveedores CP ON CP.id=CO.proveedor_id WHERE CO.ocompra='POC5-FBV';";
$result = mysqli_query($conn, $query);
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $codigo = $row['ocompra'];
    $proveedor = $row['id'];
}

// Detalle  
$query = "SELECT COD.producto_id sku,COD.cantidad FROM comex_ocompra_det COD WHERE COD.ocompra_id=4;";
$result = mysqli_query($conn, $query);
$detalle = [];
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $detalle[] = [
        "sku" => $row['sku'],
        "unidadesEnviadas" => $row['cantidad'],
        "campo01" => "",
        "campo02" => ""
    ];
}

```
#### Codigo PHP usando CURL
```php
$url = "tuUrl";
$token = "tuToken";
$inserData = [
    "usuario" => "tuUsuario", // Se deben crear credenciales SOLO para integracion 
    "password" => "tuClave",
    "ordenDeCompra" => [
        "codigo" => $codigo,
        "sitio" => "CD_PROVALTEC", 
        "proveedor" => $proveedor, 
        "tipo" => "IMPORTACION", 
        "descripcion" => "Prueba de inyeccion", 
        "listaDetalle" => $detalle
    ]
];

$conInv = curl_init($url);
$headers = [];
$headers[] = 'Content-Type:application/json';
$headers[] = "Authorization: Bearer " . $token;
curl_setopt($conInv, CURLOPT_HTTPHEADER, $headers);
curl_setopt($conInv, CURLOPT_POSTFIELDS, json_encode($inserData));
curl_setopt($conInv, CURLOPT_RETURNTRANSFER, true);
$data = curl_exec($conInv);
curl_close($conInv);

```
### Creacion de OC

Solo indicaremos los compos que son requeridos, para mayor detalle solicitar la coleccion a Impruvex

```http
  POST /api/invas/rest/os/upload
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|
| `idOs` | `string` | Hace referencia al numero de cotizacion en el sistema (cotizador)|
| `sitio` | `string` | El sitio siempre sera CD_PROVALTEC|
| `tipoOs` | `string` | **Pendiente de definir**|
| `cliente` | `string` | Rut de la empresa  |
| `fechaPactada` | `string` | Fecha de creacion|
| `nombreCliente` | `string` | Razon Social|
| `telefonoCliente` | `string` | |
| `direccionCliente` | `string` |  |
| `ciudadcliente` | `string` |  |
| `emailCliente` | `string` | |
| `destino` | `string` | **Pendiente de definir** |
| `clienteComuna` | `string` |  |
| `producto` | `string` | Id del producto |
| `unidadesOrdenadas` | `string` | Cantidad vendida |

#### Consulta MYSQL

```mysql
//  Cabecera 
$query = "SELECT C.cotizacion, C.fecha, CL.rut, CL.razon_social, IFNULL(CL.telefonos,'')telefono, IFNULL(CL.direccion,'') direccion, IFNULL(CO.descripcion,'') comuna, IFNULL(CL.email,'') email FROM cotizaciones C JOIN clientes CL ON CL.id=C.cliente_id JOIN comunas CO ON CO.id=CL.comuna WHERE C.cotizacion='111444VV';";

// Detalle  
$query = "SELECT CD.producto_id sku, CD.cantidad FROM cotizaciones_det CD WHERE cotizacion_id='112150';";
```
#### Codigo PHP usando CURL
```php
$result = mysqli_query($conn, $query);
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $os = $row['cotizacion'];
    $fecha = $row['fecha'];
    $cliente = $row['rut'];
    $nombreCliente = $row['razon_social'];
    $telefonoCliente = $row['telefono'];
    $direccionCliente = $row['direccion'];
    $ciudadcliente = $row['comuna'];
    $emailCliente = $row['email'];
}

$result = mysqli_query($conn, $query);
$detalle = [];
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $detalle[] = [
        "producto" => $row['sku'],
        "unidadesOrdenadas" => $row['cantidad']
    ];
}
$url = "tuUrl";
$token = "tuToken";
$inserData = [
    "usuario" => "tuUsuario", 
    "password" => "tuClave",
    "os" => [
        "idOs" => $os,
        "sitio" => "CD_PROVALTEC",
        "tipoOs" => "OS-PTV",
        "cliente" => $cliente,
        "fechaPactada" => $fecha,
        "nombreCliente" => $nombreCliente,
        "telefonoCliente" => $telefonoCliente,
        "direccionCliente" => mb_convert_encoding($direccionCliente, 'UTF-8', 'ISO-8859-1'), // usar utf8_encode para versiones > 7
        "ciudadcliente" => $ciudadcliente,
        "emailCliente" => $emailCliente,
        "destino" => "Definir",//Definir nombre de ruta
        "clienteRegion" => "",
        "clienteComuna" => $ciudadcliente,
        "listaTipoInsumo" => $detalle

    ]
];

$conInv = curl_init($url);
$headers = [];
$headers[] = 'Content-Type:application/json';
$headers[] = "Authorization: Bearer " . $token;
curl_setopt($conInv, CURLOPT_HTTPHEADER, $headers);
curl_setopt($conInv, CURLOPT_POSTFIELDS, json_encode($inserData));
curl_setopt($conInv, CURLOPT_RETURNTRANSFER, true);
$data = curl_exec($conInv);
curl_close($conInv);

```
### Resumen de inventario
El stock se puede consultar unitariamente o de forma global, usaremos ambas opciones. Para consumir unitariamente el stock se deben enviar ambos parametros, sku y sitio. Pero para consumir todo el stock se debe enviar solo el sitio.

### Obtener stock

```http
  POST /api/invas/rest/WmsResumenInventario/consultaInventarioSku
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|
| `sku` | `string` | id del producto.|
| `sitio` | `string` | CD_PROVALTEC |

#### Codigo PHP usando CURL
```php
$url = "tuUrl";
$token = "tuToken";
$codigo = 1359;
$inserData = [
    "usuario" => "tuUsuario",
    "password" => "tuClave",
    "listasku" => [
        [
            "sku" => $codigo,
            "sitio" => "CD_PROVALTEC"
        ]
    ]
];

$conInv = curl_init($url);
$headers = [];
$headers[] = 'Content-Type:application/json';
$headers[] = "Authorization: Bearer " . $token;
curl_setopt($conInv, CURLOPT_HTTPHEADER, $headers);
curl_setopt($conInv, CURLOPT_POSTFIELDS, json_encode($inserData));
curl_setopt($conInv, CURLOPT_RETURNTRANSFER, true);
$data = curl_exec($conInv);
$result = json_decode($data, true);
$td = $result['listainvsku'][0]['totalDisponible'];
$uo = $result['listainvsku'][0]['undsOrdenadas'];
$stkComercial = $td - $uo;
echo $stkComercial;
curl_close($conInv);
```

## Consumo de Web Services Manager

Manager posee un WS para todos sus clientes, este contiene algunos modulos del producto Time ERP. Se usaran 2 de estos modulos que son, abastecimiento y Ventas. Abastecimientos lo usaremos para inyectar la OC desde el modulo de Comex. Ventas lo usaremos para crear una Nota de venta la cual luego se transformara en una Factura para el cliente final. 

### Creacion de cabecera OC Manager 
```http
  POST /sec/prod/abastecimiento.asmx?wsdl
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `rutEmpresa` | `string` | **Requerido**. |
| `token` | `string` | **Requerido**.|
| `rutProveedor` | `string` | id del producto.|
| `fecha` | `string` |  |
| `atencion` | `string` | Usaremos el nombre de la PO |
| `tipoMoneda` | `string` | Manager usa los valores de $ EURO USD |
| `valorMoneda` | `int` |  |
| `montoDescuento` | `int` | no aplica |
| `tipoDescuento` | `int` | no aplica |
| `remitente` | `string` |  |
| `sucursal` | `int` |  |
| `tipo` | `string` |  |
| `fechaEntrega` | `string` |  |
| `cotizRef` | `int` | no aplica |
| `centroCosto` | `int` |  |
| `glosaContable` | `string` |  |
| `observacion` | `string` |  |

#### Consulta MYSQL

```mysql
//  Cabecera 
$query = "SELECT CO.ocompra,CO.moneda_id moneda,DATE_FORMAT(CO.fecha_ocompra,'%d/%m/%Y') fecha,DATE_FORMAT(CO.fecha_entrega,'%d/%m/%Y') entrega FROM comex_ocompra CO JOIN comex_proveedores CP ON CP.id=CO.proveedor_id WHERE CO.ocompra='POC5-FBV';";
```

#### Codigo PHP usando SOAP
```php
$result = mysqli_query($conn, $query);
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $oc = $row['ocompra'];
    $fecha = $row['fecha'];
    $entrega = $row['entrega'];
    $moneda = $row['moneda'];
}
switch ($moneda) {
    case 1:
        $tipoMoneda = "USD";
        break;
    case 2:
        $tipoMoneda = "EUR0";
        break;
    case 3:
        $tipoMoneda = "$";
        break;
}

$usuario = 'turutempresa;
$clave = 'tuToken';

$cliente = new SoapClient($uri);
$cabecera = array(
    'rutEmpresa' => $usuario,
    'token' => $clave,
    'rutProveedor' => '1040-5',
    'fecha' => $fecha,
    'atencion' => $oc,
    'tipoMoneda' => $tipoMoneda,
    'valorMoneda' => 0,
    'montoDescuento' => 0,
    'tipoDescuento' => 0,
    'remitente' => 'MAC',
    'sucursal' => 9,
    'tipo' => '', 
    'fechaEntrega' => $entrega,
    'cotizRef' => 0,
    'centroCosto' => 1800,
    'glosaContable' => '', 
    'observacion' => ''
);

$respuesta = $cliente->IngresaCabeceraOrdenDeCompra($cabecera);
$data = $respuesta->IngresaCabeceraOrdenDeCompraResult;
$numOC = $data->Pk;
```

### Creacion de detalle OC Manager 
```http
  POST /sec/prod/abastecimiento.asmx?wsdl
```

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `rutEmpresa` | `string` | **Requerido**. |
| `token` | `string` | **Requerido**.|
| `numOc` | `string` | numero entregado por la inyeccion de la cabecera|
| `codigoArticulo` | `string` |  |
| `cantidad` | `string` |  |
| `precio` | `string` |  |
| `cuentaContable` | `string` |  |
| `centroCosto` | `string` |  |
| `bodega` | `string` |  |
| `fechaEntrega` | `string` |  |
| `sucursal` | `string` |  |
| `tipo` | `string` |  |
| `fechaEntrega` | `string` |  |
| `terminado` | `string` |  |


#### Consulta MYSQL

```mysql
//  Detalle 
$query = "SELECT P.codigo_propio,COD.cantidad,COD.unitario FROM comex_ocompra_det COD JOIN productos P ON P.id=COD.producto_id WHERE COD.ocompra_id=4;"; // id de prueba
```

#### Codigo PHP usando SOAP
```php
$result = mysqli_query($conn, $query);
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $uri = "tuUrl";
    $usuario = 'turutempresa';
    $clave = 'tuToken';
    $cliente = new SoapClient($uri);
    $codigo = $row['codigo_propio'];
    $cantidad = $row['cantidad'];
    $unitario = $row['unitario'];
    $detalle = array(
        'rutEmpresa' => $usuario,
        'token' => $clave,
        'numOc' => $numOC,
        'codigoArticulo' => $codigo,
        'cantidad' => $cantidad,
        'precio' => $unitario,
        'cuentaContable' => 110801,
        'centroCosto' => 1008, // 10800 en la base productiva
        'bodega' => 'B1',
        'fechaEntrega' => $entrega,
        'terminado' => 0
    );
    // inyeccion de la detalle a Manager
    $respuesta = $cliente->IngresaDetalleOrdenDeCompra($detalle);
    $data = $respuesta->IngresaDetalleOrdenDeCompraResult;
    print_r($data);
}
```


## Construccion de api Provaltec

en desarrollo

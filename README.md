
# Integracion Impruvex Provaltec y Manager

Impruvex ha llegado a establecer un control exacto y llevar la logística a otro nivel en Provaltec. Pero también llego a sacudir el área de IT con una integración de ambos sistemas Legacy (cotizador, comex) a Impruvex y Legacy(cotizador, comex) a Manager. En esta documentación hablaremos y detallaremos toda la integración, estará dividida en 3 partes, primero la integración con Impruvex, la integración con Manager y como último la creación de nuestra API para que Impruvex pueda guardar datos en nuestros sistemas. 
 
 

### Temario
- Obtención de token
- Creación de OC
- Creación de OS
- Resumen de inventario
- Creación OC en Manager
- Creacion de Nota de venta en Manager
- Envíos a nuestra API


### Lenguajes usados

**Legacy:** PHP, Mysql, Bootstrap

**Api:** Node JS

![Logo](https://provaltec.samurai.cl/img/logo/logo-header-2021-03-24-14-26-05-le76317144-2.jpg)
![Logo](https://demoinvas.impruvex.com/plataforma/images/newlogo.png)
![Logo](https://www.manager.cl/wp-content/uploads/2023/07/5-1.webp)




## Consumo de Apis Impruvex

Impruvex nos ha facilitado sus Apis para inyectar la información que se ingrese en nuestro sistema, Pero nesecitamos un token de acceso el cual se entrega mediante el inicio de sesión. Es por eso que se necesitan unas credenciales de acceso. Estas credenciales fueron entregadas en la capacitación realizada por el Maestro Marcelo Fierro y es un usuario especialmente para la integración. A continuación detallo como se solicita el token.


### Obtener token

```http
  POST /api/invas/rest/usuario/loginWS
```

| Parametro | Tipo     | Descripción                |
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

Solo indicaremos los campos que son requeridos, para mayor detalle solicitar la colección a Impruvex

```http
  POST /api/invas/rest/oc/upload
```

| Parametro | Tipo     | Descripción                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|
| `codigo` | `string` | Hace referencia al número de PO en el sistema Comex|
| `sitio` | `string` | El sitio siempre será CD_PROVALTEC|
| `proveedor` | `string` |  El RUT del proveedor, en Comex no tenemos RUT, por lo que hay que agregar un campo a la tabla|
| `tipo` | `string` | El tipo es IMP, pero estamos evaluando la opción de pasar lo nacional también por el módulo de Comex, usar NAC para los nacionales |
| `descripcion` | `string` | Descripción de la compra|
| `sku` | `string` | Id del producto|
| `unidadesEnviadas` | `string` | Cantidades que se compraron|

#### Consulta MYSQL

```mysql
//  Cabecera 
 $query = "SELECT CO.ocompra,CP.identificacion,CO.proforma FROM comex_ocompra CO JOIN comex_proveedores CP ON CP.id=CO.proveedor_id WHERE CO.id=".$id.";";
        $result = mysqli_query($conn, $query);
        while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
            $codigo = $row['ocompra'];
            $proveedor = $row['identificacion'];
            $invoice = $row['proforma'];
}

// Detalle  
$query = "SELECT COD.producto_id sku,COD.cantidad FROM comex_ocompra_det COD WHERE COD.ocompra_id=".$id.";";
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
                "tipo" => "IMP", 
                "descripcion" => "OC ingresada mediante integracion",
                "campo01" => $invoice,
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
### Creacion de OS

Solo indicaremos los campos que son requeridos, para mayor detalle solicitar la colección a Impruvex

```http
  POST /api/invas/rest/os/upload
```

| Parametro | Tipo     | Descripción                |
| :-------- | :------- | :------------------------- |
| `usuario` | `string` | **Requerido**. |
| `password` | `string` | **Requerido**.|
| `idOs` | `string` | Hace referencia al número de cotización en el sistema (cotizador)|
| `sitio` | `string` | El sitio siempre sera CD_PROVALTEC|
| `tipoOs` | `string` | OV para pedidos normales y OSEXPRESS para los pedidos presenciales|
| `cliente` | `string` | RUT de la empresa  |
| `fechaPactada` | `string` | Fecha de creacion|
| `nombreCliente` | `string` | Razon Social|
| `telefonoCliente` | `string` | |
| `direccionCliente` | `string` |  |
| `ciudadcliente` | `string` |  |
| `emailCliente` | `string` | |
| `destino` | `string` | RUTA_PRINCIPAL,RUTA_ECOMMERCE,RUTA_ECOMMERCE |
| `clienteComuna` | `string` |  |
| `producto` | `string` | Id del producto |
| `unidadesOrdenadas` | `string` | Cantidad vendida |

#### Consulta MYSQL

```mysql
//  Cabecera 
	$query = "SELECT C.cotizacion, DATE_FORMAT(C.fecha_req,'%Y/%m/%d') fecha, CL.rut, CL.razon_social,C.origen_id, IFNULL(CL.telefonos,'')telefono, IFNULL(CL.direccion,'') direccion, IFNULL(CO.descripcion,'') comuna, IFNULL(C.email_aviso,'') email,C.despacho_retiro,C.cond_despacho,C.req_certificado FROM cotizaciones C LEFT JOIN clientes CL ON CL.id=C.cliente_id LEFT JOIN comunas CO ON CO.id=CL.comuna WHERE C.id=" . trim($id) . ";";
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
		$idOrigen = $row['origen_id'];
		$despachoRetiro = $row['despacho_retiro'];
		$condicionDespacho = $row['cond_despacho'];
		$certificado = $row['req_certificado'];
	}

// Detalle  
$query = "SELECT CD.producto_id sku, CD.cantidad FROM cotizaciones_det CD WHERE cotizacion_id='112150';";
$result = mysqli_query($conn, $query);
$detalle = [];
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $detalle[] = [
        "producto" => $row['sku'],
        "unidadesOrdenadas" => $row['cantidad']
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
		"os" => [
			"idOs" => $os,
			"sitio" => "CD_PROVALTEC",
			"tipoOs" => $origen,
			"cliente" => $cliente,
			"fechaPactada" => $fecha,
			"nombreCliente" => $nombreCliente,
			"telefonoCliente" => $telefonoCliente,
			"direccionCliente" => mb_convert_encoding($direccionCliente, 'UTF-8', 'ISO-8859-1'), // usar utf8_encode para versiones 7.1
			"ciudadcliente" => $ciudadcliente,
			"emailCliente" => $emailCliente,
			"destino" => $destino,
			"campo01" => trim($condicionDespacho),
			"campo02" => $despacho,
			"campo03" => $cert,
			"clienteRegion" => "",
			"clienteComuna" => $ciudadcliente,
			"listaTipoInsumo" => $detalle

		]
	];

	//Conexion a API invas mediante CURL
	$conInv = curl_init($url);
	$headers = [];
	$headers[] = 'Content-Type:application/json';
	$headers[] = "Authorization: Bearer " . $token;
	curl_setopt($conInv, CURLOPT_HTTPHEADER, $headers);
	curl_setopt($conInv, CURLOPT_POSTFIELDS, json_encode($inserData));
	curl_setopt($conInv, CURLOPT_RETURNTRANSFER, true);
	$data = curl_exec($conInv);

```
### Resumen de inventario
El stock se puede consultar unitariamente o de forma global, usaremos ambas opciones. Para consumir unitariamente el stock se deben enviar ambos parámetros, sku y sitio. Pero para consumir todo el stock se debe enviar solo el sitio.

### Obtener stock

```http
  POST /api/invas/rest/WmsResumenInventario/consultaInventarioSku
```

| Parametro | Tipo     | Descripción                |
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

Manager posee un WS para todos sus clientes, este contiene algunos módulos del producto Time ERP utilizaremos 2 de estos módulos que son, abastecimiento y Ventas. En Abastecimientos inyectaremos la OC desde el módulo de Comex. Ventas lo emplearemos para crear una Nota de venta, la cual luego se transformara en una Factura para el cliente final. 

### Creacion de cabecera OC Manager 
```http
  POST /sec/prod/abastecimiento.asmx?wsdl
```

| Parametro | Tipo     | Descripción                |
| :-------- | :------- | :------------------------- |
| `rutEmpresa` | `string` | **Requerido**. |
| `token` | `string` | **Requerido**.|
| `rutProveedor` | `string` | id del producto.|
| `fecha` | `string` |  |
| `atencion` | `string` | utilizaremos el nombre de la PO |
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
$result = mysqli_query($conn, $query);
while ($row = mysqli_fetch_array($result, MYSQLI_ASSOC)) {
    $oc = $row['ocompra'];
    $fecha = $row['fecha'];
    $entrega = $row['entrega'];
    $moneda = $row['moneda'];
}
```

#### Codigo PHP usando SOAP
```php

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

| Parametro | Tipo     | Descripción                |
| :-------- | :------- | :------------------------- |
| `rutEmpresa` | `string` | **Requerido**. |
| `token` | `string` | **Requerido**.|
| `numOc` | `string` | número entregado por la inyección de la cabecera|
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
$query = "SELECT P.codigo_propio,COD.cantidad,COD.unitario FROM comex_ocompra_det COD JOIN productos P ON P.id=COD.producto_id WHERE COD.ocompra_id=4;";
$result = mysqli_query($conn, $query);
```

#### Codigo PHP usando SOAP
```php

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
        'centroCosto' => 1008,
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

### Recepción de ASN
```http
  POST /api/impruvex/asn/
```
Impruvex nos envía un JSON con mucha información sobre la llegada de mercadería a Provaltec. La API recibe esta información y la procesa en la tabla recepciones_invas, la cual tiene los siguientes campos.

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `fecha_recepcion` | `date` `NULL`  | Fecha de recepción de la ASN |
| `ocompra` | `varchar(30)` `NULL`| Numero de oc|
| `producto_id` | `int(10) unsigned` `NULL` | Código del producto|
| `cantidad` | `smallint(5) unsigned` `NULL` | Unidades recibidas  |
| `estado` | `tinyint(3) unsigned` `NOT NULL` | estado usado para notificación  |

#### Codigo Node js
```js
export const verificarAsn = async (req, res) => {
    await req.getConnection(async (error, conexion) => {
        if (error) return res.send(error)
        const post = req.body
        const subPost = req.body.listaCajas // get nested
        await subPost.forEach(e => {
            let sql = 'INSERT INTO recepciones_invas VALUES (?,?,?,?,?,?);'
            conexion.query(sql, [undefined, post.fecharecep, post.numoc, e.sku, e.unidades, 0], (err, rows) => {
                if (err) throw err 
            })
        })
        await conexion.query('SELECT COUNT(id) row FROM recepciones_invas WHERE ocompra=?', [post.numoc], (err, rows) => {
            if (err) throw err 
            res.json({
                "data": true,
                "DIresult": "Success",
                "DImsg": post.numoc,
                "messaje": "Datos Recibidos",
                "cantidadProductos": rows[0].row

            })
        })
    })
}
```
### Despacho Express
```http
  POST /api/impruvex/despachoExpress/
```
El Despacho Express nos avisa que ya fue procesado y está listo para retirarlo o bien despachar el pedido, esta información la guardamos en la tabla despachos_invas.

| Parametro | Tipo     | Descripcion                |
| :-------- | :------- | :------------------------- |
| `fecha_despacho` | `date` `NULL`  | Fecha del despacho |
| `cotizacion` | `varchar(30)` `NULL`| Numero de OS (cotizacion)|
| `producto_id` | `int(10) unsigned` `NULL` | Código del producto|
| `cantidad` | `smallint(5) unsigned` `NULL` | Unidades recibidas  |
| `estado` | `tinyint(3) unsigned` `NOT NULL` | estado usado para notificación  |

#### Codigo Node js
```js
export const despachoExpress = async (req, res) => {
    const post = req.body.listaLpnDestino
    var fecha = new Date().getFullYear() + '-' + ("0" + (new Date().getMonth() + 1)).slice(-2) + '-' + ("0" + new Date().getDate()).slice(-2)
    await req.getConnection(async (error, conexion) => {
        if (error) return res.send(error)
        await post.forEach(e => {
            let sql = 'INSERT INTO despachos_invas VALUES (?,?,?,?,?,?);'
            conexion.query(sql, [undefined, fecha, e.ordendesalida, e.sku, e.unidades, 0], (err, rows) => {
                if (err) throw err 
            })
        })
        res.json({
            "Results": [{
                    "data": true,
                    "ordendesalida": post[0].ordendesalida,
                    "DIresult": "Success",
                    "DImsg": "179116",
                    "messaje": "Datos Recibidos",
                }]
        })
    })
}
```

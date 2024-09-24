async abrirVistaImpresionTablaR() {
      let arrayFiltrado = [];
      this.dialogImpresion = false;
      this.dialogOpcionesTablaR = false;

      // if (this.itemsFiltrados.length === 0) {
      //   arrayFiltrado = this.itemsNombre[this.selectedTab].pedidos;
      // } else {
      //   arrayFiltrado = this.itemsFiltrados;
      // }

      arrayFiltrado = this.itemsNombre[this.selectedTab].pedidos;

      console.log(arrayFiltrado);

      // Se obtienen las localizaciones de todos los pedidos
      const localizacionesCache = new Map(); // Map para almacenar localizaciones ya obtenidas

      // Funcion para solicitar la localizacion o obtenerla de la cache
      const getCachedLocalizacion = async (idCliente) => {
        if (!localizacionesCache.has(idCliente)) {
          // Si la localización no ha sido obtenida aún, hacer la solicitud
          const localizacionPromise = this.getLocalizacion(idCliente);
          localizacionesCache.set(idCliente, localizacionPromise); // Guardar la promesa temporalmente
          const localizacion = await localizacionPromise;
          // Actualizar el Map con la localización resuelta
          localizacionesCache.set(idCliente, {
            latitud: localizacion.latitud || "",
            longitud: localizacion.longitud || "",
            dia: localizacion.dia,
          });
        }
        // Usar la localización almacenada en el cache
        return localizacionesCache.get(idCliente);
      };

      const arrayLocalizaciones = await Promise.all(
        arrayFiltrado.map(async (item) => {
          const localizacion = await getCachedLocalizacion(item.idCliente);
          const nuevoItem = {
            id: item.id,
            latitud: localizacion.latitud,
            longitud: localizacion.longitud,
            dia: localizacion.dia,
            cliente: item.cliente,
            total: item.total,
            facturas: item.facturas,
            contado: item.contado,
            zona: item.zona,
            fecha_creacion: item.fecha_creacion,
            mensajero: item.mensajero,
            estado: item.estado,
          };
          return nuevoItem;
        })
      );

      // console.log(arrayLocalizaciones);

      let nuevoArray = [];

      // Filtrar los pedidos por dia actual si es que se requiere
      if (this.checkboxDia) {
        nuevoArray = arrayLocalizaciones.filter(
          (item) => item.dia === this.currentDay
        );
      } else {
        nuevoArray = arrayLocalizaciones;
      }
      /*
      //nueva implementacion de asignar mensajero seleccionado a los pedidos filtrados por estado
      nuevoArray.forEach((pedido) => {
        if (this.estadosSelected.includes(pedido.estado)) {
          pedido.mensajero = this.mensajero || "Mensajero no asignado";
        }
      });*/

      // Separamos los pedidos por estado para mostrarlos en la tabla
      let pedidosCreados = [];
      let pedidosRevisionCredito = [];
      let pedidosConfirmacion = [];
      let pedidosFacturacion = [];
      let pedidosPreparadosAlmacen = [];
      let pedidosPreparados = [];
      let pedidosEnRuta = [];

      if (this.estadosSelected.includes("Pedido creado")) {
        pedidosCreados = nuevoArray.filter(
          (item) => item.estado === "Pedido creado"
        );
      }

      if (this.estadosSelected.includes("Revisión de crédito")) {
        pedidosRevisionCredito = nuevoArray.filter(
          (item) => item.estado === "Revisión de crédito"
        );
      }

      if (this.estadosSelected.includes("Confirmación")) {
        pedidosConfirmacion = nuevoArray.filter(
          (item) => item.estado === "Confirmación"
        );
      }

      if (this.estadosSelected.includes("Facturación")) {
        pedidosFacturacion = nuevoArray.filter(
          (item) => item.estado === "Facturación"
        );
      }

      if (this.estadosSelected.includes("Preparación en almacén")) {
        pedidosPreparadosAlmacen = nuevoArray.filter(
          (item) => item.estado === "Preparación en almacén"
        );
      }

      if (this.estadosSelected.includes("Preparado")) {
        pedidosPreparados = nuevoArray.filter(
          (item) => item.estado === "Preparado"
        );
      }

      if (this.estadosSelected.includes("En ruta de entrega")) {
        pedidosEnRuta = nuevoArray.filter(
          (item) => item.estado === "En ruta de entrega"
        );
      }

      // Crear una ventana emergente
      const ventana = window.open("", "_blank");

      // Construir el contenido HTML de la vista previa de impresión
      let contenidoTablaCreados = "";
      let contenidoTablaRevisionCredito = "";
      let contenidoTablaConfirmacion = "";
      let contenidoTablaFacturacion = "";
      let contenidoTablaPreparadosAlmacen = "";
      let contenidoTablaPreparados = "";
      let contenidoTablaEnRuta = "";
      const contenidoHTML = `
      <!doctype html>
      <html lang="es">

      <head>

          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>Pedidos</title>
          <!-- Enlace al archivo CSS de Bootstrap -->
          <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"
              integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
          <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
          <style>
              @import url('https://fonts.googleapis.com/css2?family=Arvo:wght@700&display=swap');
              @page { size: landscape; } 

              body {
                  font-family: Arial, sans-serif;
                  display: flex;
                  flex-direction: column;
                  height: 100vh;
                  margin: 0;
              }

              .content {
                  flex: 1;
              }

              .table-table {
                  width: 100%;
                  border-collapse: collapse;
                  font-size: 11px;
              }

              .tabla-th {
                  background-color: #f2f2f2;
                  padding: 8px;
                  text-align: left;
              }

              .tabla-td {
                  padding: 8px;
                  text-align: left;
              }

              .datos-table {
                  width: 100%;
                  border-collapse: collapse;
              }

              .datos-th {
                  padding: 8px;
                  font-size: 10px;
                  text-align: left;
              }

              .datos-td {
                  padding: 8px;
                  font-size: 14px;
                  text-align: left;
              }

              .comment-column {
                  width: 200px;
              }

              .comment-column-data {
                  width: 200px;
                  height: 50px;
              }

              @media print {
                  .tabla-th,
                  .tabla-th {
                      background-color: #f2f2f2 !important;
                  }
                  th, td {
                    border: 1px solid black;
                    padding: 8px;
                    text-align: left;
                  }
                    img .logo {
                        display: block;
                    }
                      .header {
                        background-color: #5480DF !important;
                        color: #000 !important;
                      }
              }

.header {
  padding-top: 15px;
  padding-bottom: 15px;
  background: #5480DF;
  border-bottom: 3px solid #ccc;
  font-size: 12px;
  font-weight: 600;
  text-align: center;
  color: #fff;
  border: 1px solid black;
}

.logo {
position: absolute;
  width: 130px;
  height: 100px;
  margin-left: 20px;
  margin-top: 10px;
  /* posicionar a la izquierda */
  left: 0;
  /* posicionar arriba */
  top: 0;
  filter: drop-shadow(0 0 3px white);
}
.full-width-row td {
            padding: 15px;
            background-color: #f9f9f9;
            text-align: center;
            border-top: 1px solid #ddd;
            /* Establece el color del texto y el fondo para el contenido adicional */
            font-weight: bold;
        }

         .firma-line {
            border-top: 1px solid black;
            margin-top: 60px;
            width: 100%;
            text-align: center;
        }

        .firma-cell {
            padding-top: 80px;
            text-align: center;
        }
            .total {
            text-align: right;
            font-weight: bold;
        }
          </style>
      </head>

      <body class="d-flex">

          <div class="d-flex flex-column">
              <div class="header">
                <img class="logo" src="https://prostasa.com/uploads/1/4/3/4/143499628/logo-prostasa.webp" alt="Logo de la empresa">
                GRUPO PROSTASA DE MEXICO S.A. DE C.V. <br>
               <br> Calle Lago de Opacarai No. 3143-B Colonia Lomas Del Boulevard <br>
              Culiacan Sinoloa, C.P 80120 <br>
              Tel: 667 275487 y 667 2840927</div>
              <table class="table-table table table-bordered table-sm">
                  <thead>
                      <tr>
                          <th class="tabla-th">ID</th>
                          <th class="tabla-th">Fecha pedido</th>
                          <th class="tabla-th">Facturas</th>
                          <th class="tabla-th">Cliente</th>
                          <th class="tabla-th">Importe</th>
                          <th class="tabla-th">Crédito/Contado</th>
                          <th class="tabla-th">Dia</th>
                          <th class="tabla-th ${
                            this.checkboxLatitudLongitud
                              ? " comment-column"
                              : ""
                          }">Observaciones</th>
                      </tr>
                  </thead>
                  <tbody>`;

      if (pedidosCreados.length > 0) {
        let totalSuma = 0;
        contenidoTablaCreados = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos creados</th>`
                          : `<th colspan="12">Pedidos creados</th>`
                      }
                    </tr>
                     ${pedidosCreados
                       .map((pedido) => {
                         // Sumar cada total al totalSuma
                         totalSuma += parseFloat(pedido.total);

                         return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                       })
                       .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosRevisionCredito.length > 0) {
        let totalSuma = 0;
        contenidoTablaRevisionCredito = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos en revisión de crédito</th>`
                          : `<th colspan="12">Pedidos en revisión de crédito</th>`
                      }
                    </tr>
                      ${pedidosRevisionCredito
                        .map((pedido) => {
                          // Sumar cada total al totalSuma
                          totalSuma += parseFloat(pedido.total);

                          return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                        })
                        .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosConfirmacion.length > 0) {
        let totalSuma = 0;
        contenidoTablaConfirmacion = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos en confirmación</th>`
                          : `<th colspan="12">Pedidos en confirmación</th>`
                      }
                    </tr>
                      ${pedidosConfirmacion
                        .map((pedido) => {
                          // Sumar cada total al totalSuma
                          totalSuma += parseFloat(pedido.total);

                          return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                        })
                        .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosFacturacion.length > 0) {
        let totalSuma = 0;
        contenidoTablaFacturacion = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos en facturación</th>`
                          : `<th colspan="12">Pedidos en facturación</th>`
                      }
                    </tr>
                      ${pedidosFacturacion
                        .map((pedido) => {
                          // Sumar cada total al totalSuma
                          totalSuma += parseFloat(pedido.total);

                          return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                        })
                        .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosPreparadosAlmacen.length > 0) {
        let totalSuma = 0;
        contenidoTablaPreparadosAlmacen = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos preparados en almacén</th>`
                          : `<th colspan="12">Pedidos preparados en almacén</th>`
                      }
                    </tr>
                      ${pedidosPreparadosAlmacen
                        .map((pedido) => {
                          // Sumar cada total al totalSuma
                          totalSuma += parseFloat(pedido.total);

                          return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                        })
                        .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosPreparados.length > 0) {
        let totalSuma = 0;
        contenidoTablaPreparados = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos preparados</th>`
                          : `<th colspan="12">Pedidos preparados</th>`
                      }
                    </tr>
                      ${pedidosPreparados
                        .map((pedido) => {
                          // Sumar cada total al totalSuma
                          totalSuma += parseFloat(pedido.total);

                          return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                        })
                        .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>
        <tr class="full-width-row">
    
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }

      if (pedidosEnRuta.length > 0) {
        let totalSuma = 0; //inicializa la suma total
        contenidoTablaEnRuta = `
                    <tr class="table-secondary">
                      ${
                        this.checkboxLatitudLongitud
                          ? `<th colspan="10">Pedidos en ruta de entrega</th>`
                          : `<th colspan="12">Pedidos en ruta de entrega</th>`
                      }
                    </tr>
                       ${pedidosEnRuta
                         .map((pedido) => {
                           // Sumar cada total al totalSuma
                           totalSuma += parseFloat(pedido.total);

                           return `
                <tr>
                    <td class="tabla-td">${pedido.id}</td>
                    <td class="tabla-td">${this.formatoFecha(
                      pedido.fecha_creacion
                    )}</td>
                    <td class="tabla-td">${pedido.facturas}</td>
                    <td class="tabla-td">${pedido.cliente}</td>
                    <th class="tabla-th">$${pedido.total.toFixed(2)}</th>
                    <td class="tabla-td">${
                      pedido.contado ? "Contado" : "Credito"
                    }</td>
                    <td class="tabla-td">${pedido.dia}</td>
                    <td class="tabla-td">${" "}</td>
                </tr>
            `;
                         })
                         .join("")}
        <tr>
            <td colspan="4" class="total">Total:</td>
            <td class="total">$${totalSuma.toFixed(2)}</td>
            <td colspan="5"></td>
        </tr>

        <tr class="full-width-row">
   
    <td colspan="12" class="total">SON: ${(() => {
      const numeroATexto = (numero) => {
        const unidades = [
          "",
          "UNO",
          "DOS",
          "TRES",
          "CUATRO",
          "CINCO",
          "SEIS",
          "SIETE",
          "OCHO",
          "NUEVE",
        ];
        const decenas = [
          "",
          "",
          "VEINTE",
          "TREINTA",
          "CUARENTA",
          "CINCUENTA",
          "SESENTA",
          "SETENTA",
          "OCHENTA",
          "NOVENTA",
        ];
        const especiales = [
          "DIEZ",
          "ONCE",
          "DOCE",
          "TRECE",
          "CATORCE",
          "QUINCE",
          "DIECISEIS",
          "DIECISIETE",
          "DIECIOCHO",
          "DIECINUEVE",
        ];
        const centenas = [
          "",
          "CIENTO",
          "DOSCIENTOS",
          "TRESCIENTOS",
          "CUATROCIENTOS",
          "QUINIENTOS",
          "SEISCIENTOS",
          "SETECIENTOS",
          "OCHOCIENTOS",
          "NOVECIENTOS",
        ];

        if (numero === 0) return "CERO";
        if (numero === 100) return "CIEN";

        let texto = "";

        // Manejar miles
        if (numero >= 1000) {
          const miles = Math.floor(numero / 1000);
          if (miles === 1) {
            texto += "MIL ";
          } else {
            texto += numeroATexto(miles) + " MIL ";
          }
          numero = numero % 1000;
        }

        // Manejar centenas
        if (numero > 99) {
          texto += centenas[Math.floor(numero / 100)] + " ";
          numero = numero % 100;
        }

        // Manejar decenas y unidades
        if (numero > 19) {
          texto +=
            decenas[Math.floor(numero / 10)] +
            (numero % 10 === 0 ? "" : " Y " + unidades[numero % 10]);
        } else if (numero > 9) {
          texto += especiales[numero - 10];
        } else {
          texto += unidades[numero];
        }

        return texto.trim();
      };

      const total = totalSuma.toFixed(2).split(".");
      const parteEntera = parseInt(total[0]);
      const parteDecimal = parseInt(total[1]);

      return `${numeroATexto(parteEntera)} PESOS ${
        parteDecimal > 0 ? `CON ${numeroATexto(parteDecimal)} CENTAVOS` : ""
      }`;
    })()}</td>
    
</tr>
    `;
      }
      const contenidoHTML2 = `
      <tr class="full-width-row">
        <td colspan="12">
          YO: <u>${
            this.mensajeroSelected.toUpperCase()
              ? this.mensajeroSelected.toUpperCase()
              : "Mensajero no asignado".toUpperCase()
          }</u>: DEBO Y ME HAGO RESPONSABLE POR EL RESGUARDO, COBRANZA Y/O EXTRAVIO DE LAS FACTURAS
                  ORIGINALES, AQUI RELACIONADAS Y ME COMPROMETO A ENTREGAR EL PAGO O DEVOLUCION DE LAS MISMA EN TIEMPO Y FORMA, EN CASO DE QUE NO SEA PAGADA
                  POR EL CLIENTE A SU VENCIMIENTO, PAGARE INCONDICIONALMENTE A LA ORDEN DE GRUPO PROSTASA DE MEXICO S.A. DE C.V.
          </td>

      </tr>

            <tr>
            <!-- Celda de firma izquierda -->
            <td class="firma-cell" colspan="3">
                <div class="firma-line"></div>
                <div>${this.currentUser.nombre.toUpperCase()}</div>
                <div>ENTREGA</div>
            </td>

            <!-- Celda vacía en el medio si es necesario -->
            <td colspan="3"></td>

           <!-- Celda de firma derecha -->
            <td class="firma-cell" colspan="3">
                <div class="firma-line"></div>
                <div>${
                  this.mensajeroSelected.toUpperCase()
                    ? this.mensajeroSelected.toUpperCase()
                    : "Mensajero no asignado".toUpperCase()
                }</div>
                <div>ACEPTO</div>
            </td>
        </tr>
                  </tbody>
              </table>
          </div>
          
      </body>

      </html>`;

      // Escribir el contenido HTML en la ventana emergente
      ventana.document.open();
      ventana.document.write(contenidoHTML);
      ventana.document.write(contenidoTablaCreados);
      ventana.document.write(contenidoTablaRevisionCredito);
      ventana.document.write(contenidoTablaConfirmacion);
      ventana.document.write(contenidoTablaFacturacion);
      ventana.document.write(contenidoTablaPreparadosAlmacen);
      ventana.document.write(contenidoTablaPreparados);
      ventana.document.write(contenidoTablaEnRuta);
      ventana.document.write(contenidoHTML2);
      ventana.document.close();

      // Imprimir la ventana emergente
      ventana.print();
    },
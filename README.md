/**
 * PROYECTO: inegi
 * DOCUMENTACIÓN:
 * 1. Conexión a la hoja mediante funciones base seguras.
 * 2. Filtrado de datos del DENUE utilizando lógica de arreglos.
 * 3. Cálculo de proximidad GPS con fórmula de Haversine.
 */

// --- FUNCIONES BASE (CORREGIDAS PARA EVITAR ERROR NULL) ---

function leerCelda(hoja, fila, columna) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(hoja) || ss.getSheets()[0];
  
  // VALIDACIÓN: Si fila o columna son nulos, evitar el error de ejecución
  var f = fila || 1; 
  var c = columna || 1;
  
  return sheet.getRange(f, c).getValue();
}

function leerCeldas(hoja, rango) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(hoja) || ss.getSheets()[0];
  
  // Si el rango no se define, lee toda la hoja por defecto
  var r = rango || "A:AO";
  return sheet.getRange(r).getValues();
}

function setCelda(hoja, fila, columna, valor) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(hoja) || ss.getSheets()[0];
  
  var f = fila || 1;
  var c = columna || 1;
  
  sheet.getRange(f, c).setValue(valor);
}

// --- FUNCIONES DEL TRABAJO ---

/**
 * 1. ls(arg): Busca por contacto
 */
function ls(arg) {
  var filas = leerCeldas("", "A:AO").slice(1);
  
  var resultado = filas.filter(function(f) {
    var tel = f[34] !== ""; // Columna AI
    var cor = f[35] !== ""; // Columna AJ
    var web = f[36] !== ""; // Columna AK
    
    if (arg === 't') return tel;
    if (arg === 'w') return web;
    if (arg === 'c') return cor;
    if (arg === 'a') return tel && cor && web;
    return false;
  });
  
  Logger.log("Resultados ls: " + resultado.length);
  return resultado;
}

/**
 * 2. ls_v: Busca por vialidad
 */
function ls_v(tipoVialidad, nombreVialidad) {
  var filas = leerCeldas("", "A:AO").slice(1);
  
  var resultado = filas.filter(function(f) {
    var t = f[6] ? f[6].toString().toUpperCase() : "";
    var n = f[7] ? f[7].toString().toUpperCase() : "";
    return t === tipoVialidad.toUpperCase() && n.includes(nombreVialidad.toUpperCase());
  });
  
  Logger.log("Resultados ls_v: " + resultado.length);
  return resultado;
}

/**
 * 3. lsGPS: Busca los 5 más cercanos
 */
function lsGPS(latitud, longitud) {
  var filas = leerCeldas("", "A:AO").slice(1);
  var R = 6371; // Radio KM

  var cercanos = filas.map(function(f) {
    var lat2 = parseFloat(f[38]);
    var lon2 = parseFloat(f[39]);
    
    if (isNaN(lat2) || isNaN(lon2)) return null;

    var dLat = (lat2 - latitud) * Math.PI / 180;
    var dLon = (lon2 - longitud) * Math.PI / 180;
    var a = Math.sin(dLat/2) * Math.sin(dLat/2) +
            Math.cos(latitud * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
            Math.sin(dLon/2) * Math.sin(dLon/2);
    var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    
    return { reg: f, dist: R * c };
  })
  .filter(function(i) { return i !== null && i.dist <= 3; })
  .sort(function(a, b) { return a.dist - b.dist; })
  .slice(0, 5);

  return cercanos.map(function(i) { return i.reg; });
}

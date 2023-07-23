<style>
video {
  width: 100%;
  height: auto;
}
canvas {
  display: none;
}

#customers {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

#customers td, #customers th {
  border: 1px solid #ddd;
  padding: 8px;
}

#customers tr:nth-child(even){background-color: #f2f2f2;}

#customers tr:hover {background-color: #ddd;}

#customers th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #04AA6D;
  color: white;
}
</style>

<template>
  <div id="excel" class="row items-center" style="height: 20vh; padding:10px; display:none">
    <div class="col-12 text-center" style="font-size:30px;">    
      <table id="customers" style="border: 1 solid;  overflow:auto"></table>
      &nbsp;
    </div>
    <div class="col-12 text-center">    
        <q-btn id="voltar" @click="baixar()" color="primary" icon="download" label="BAIXAR" class="full-width" size="lg"/>
        &nbsp;
        
        <q-btn id="voltar" @click="voltar()" color="primary" icon="home" label="VOLTAR" class="full-width" size="lg"/>
        &nbsp;
        <q-btn id="removertudo" @click="removertudo()" color="secondary" icon="delete" label="REMOVER TUDO" class="full-width" size="lg"/>
    </div>
  </div>



  <div id="pagina" class="row items-center" style="height: 70vh">
      <div id="div_esconde" class="col-12 text-center" style="font-size:30px;">
        &nbsp;
        <div class="col-6 text-center">
          <span class="h6">PRODUTO:</span>
        </div>
        <div class="col-6 text-center">
          <span class="h6"><input id="produto" style="min-width:90%" @input="event => text = event.target.value" ></span>
        </div>
        <div class="col-6 text-center">
          <span class="h6">VALOR:</span>
        </div>
        <div class="col-6 text-center">
          <span class="h6"><input id="valor" style="min-width:90%" @input="event => number = event.target.value" ></span>
        </div>
        <div class="col-6 text-center">
          <span class="h6">EAN:</span>
        </div>
        <div class="col-6 text-center">
          <span class="h6"><input id="ean" style="min-width:90%" @input="event => number = event.target.value" ></span>
        </div>
        &nbsp;
      </div>
      


    <div class="col-12 text-center q-pa-sm ">
      <q-btn color="primary" icon="camera_alt" label="Ler Código de Barras"
        class="full-width" size="lg" @click="iniciarLeitor()"
        v-show="cameraStatus === 0"/>
      <div class="text-h6" v-if="code"></div>
      <div id="scan" v-show="cameraStatus === 1"></div>
      <q-page-sticky position="bottom-right" :offset="[18, 18]">
        <q-btn  icon="cancel" color="negative" label="Fechar" v-show="cameraStatus === 1"
        @click="onStop" />
      </q-page-sticky>
      &nbsp;
      <q-btn id="btsalvar" @click="salvar()" color="primary" icon="save" label="SALVAR"
        class="full-width" size="lg"/>
      &nbsp;
      <q-btn id="download" @click="ver()" color="secondary" icon="download" label="VER ITENS"
        class="full-width" size="lg"/>
    </div>
  </div>
</template>

<script>
import Quagga from 'quagga'
  var lista =[];

  export default {
    data () {
      return {
        code: '',
        dialog: false,
        cameraStatus: 0
      }
    },
    methods: {
      
      iniciarLeitor () {
        this.esconder (true);
        this.cameraStatus = 1
        Quagga.init({
        inputStream: {
          name: 'Live',
          type: 'LiveStream',
          target: document.querySelector('#scan')
        },
        frequency: 10,
        decoder: {
          readers: [
            'ean_reader'
          ],
          multiple: false
        }
        }, (err) => {
        if (err) {
          alert(err)
          return
        }
        console.log('Initialization finished. Ready to start')
        Quagga.start()
        Quagga.onDetected(this.onDetected)
      })
      },
      onDetected (data) {
        this.code = data.codeResult.code
        document.getElementById("ean").value = this.code;
        this.cameraStatus = 0
        this.onStop()
      },
      onStop () {
        this.esconder (false);
        Quagga.stop()
        this.cameraStatus = 0
      },
      salvar () {
        var item = {};
        item.produto  = document.getElementById("produto").value;
        item.ean  = document.getElementById("ean").value;
        item.valor  = document.getElementById("valor").value;

        if(localStorage.getItem("lista") != null){
          lista = JSON.parse(localStorage.getItem("lista"))
        }
        lista.push(item);
        localStorage.setItem("lista", JSON.stringify(lista));

        alert('PRODUTO SALVO COM SUCESSO! ')
        document.getElementById("produto").value = '';
        document.getElementById("ean").value = '';
        document.getElementById("valor").value = '';

      },
      esconder (esconde){
        if(esconde){
          document.getElementById("ean").value = '';
          document.getElementById("btsalvar").style.display = "none";
          document.getElementById("div_esconde").style.display = "none";
          document.getElementById("download").style.display = "none";
        } else{
          document.getElementById("ean").value = this.code;
          document.getElementById("btsalvar").style.display = "block";
          document.getElementById("div_esconde").style.display = "block";
          document.getElementById("download").style.display = "block";
        }
      },

      baixar(){
        lista = JSON.parse(localStorage.getItem("lista"));
        var item = '<td>EAN<td>PRODUTO<td>VALOR';

        var customers = '';

        for(var i = 0; i < lista.length; i++){
          item = item + '<tr>';
          item = item + '<td>' + lista[i].ean + '</td>';
          item = item + '<td>' + lista[i].produto + '</td>';
          item = item + '<td>' + lista[i].valor + '</td>';
          item = item + '</tr>';
        }

        var uri = 'data:application/vnd.ms-excel;base64,'
          , template = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40"><head><meta charset="utf-8"> </head><body><table>{table}</table></body></html>'
          , base64 = function(s) { return window.btoa(unescape(encodeURIComponent(s))) }
          , format = function(s, c) { return s.replace(/{(\w+)}/g, function(m, p) { return c[p]; }) }
          var ctx = {worksheet: name || 'Planilha', table: item}
          window.location.href = uri + base64(format(template, ctx));
      },

      ver(){
        lista = JSON.parse(localStorage.getItem("lista"));
        var item = '<td>EAN<td>PRODUTO<td>VALOR';

        var customers = '';

        for(var i = 0; i < lista.length; i++){
          item = item + '<tr>';
          item = item + '<td>' + lista[i].ean + '</td>';
          item = item + '<td>' + lista[i].produto + '</td>';
          item = item + '<td>' + lista[i].valor + '</td>';
          item = item + '</tr>';
        }
        customers = customers + item;
        document.getElementById("pagina").style.display = 'none';
        document.getElementById("excel").style.display = 'block';
        document.getElementById("customers").innerHTML = customers;

        
      },

      voltar(){
        document.getElementById("pagina").style.display = 'block';
        document.getElementById("excel").style.display = 'none';
      },

      removertudo(){
        lista = [];
        localStorage.setItem("lista", JSON.stringify(lista));
        document.getElementById("customers").innerHTML = '<td>EAN<td>PRODUTO<td>VALOR';
      }
  }
}

</script>



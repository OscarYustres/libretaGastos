<template>
  <div v-if='logon' id='app' class='container'>
    <div class='text-primary'>
      <h1>Lista de gastos</h1>
    </div>
    <div class='d-block border rounded text-primary lead p-2 mt-3'>
      <div class="row">
      <div class='col'>
        Nombre del gasto
      </div>
      <div class='col'>
        <input v-model='inputNombre' class='text-primary' type='text'>
      </div>
      </div>
      <div class="row">
      <div class='col'>
        Monto del gasto
      </div>
      <div class='col'>
        <input v-model='inputMonto' class='text-primary' type='text'>
      </div>
      </div>
      <div class="row">
       <div class='col'>
        Tipo de gasto
      </div>
      <div class='col'>
        <input v-model='inputTipo' class='text-primary' type='text'>
      </div>
      </div>
      <div class='col'>
        <div id='agregar' class='btn btn-primary' 
            v-on:click='manejarClick($event)'>Agregar gasto</div>
      </div>
    </div>
     
    <!-- Filtros -->
      <div>
        <button v-bind:class='b_todos' id='todos' 
          v-on:click='filtrar($event)'>
          Todos
        </button>
        <button v-bind:class='b_hogar + " mr-2"' id='hogar' 
          v-on:click='filtrar($event)'>
          Hogar
        </button>
        <button v-bind:class='b_trabajo + " mr-2"' id='trabajo' 
          v-on:click='filtrar($event)'>
          Trabajo
        </button>
        <button v-bind:class='b_carros' id='carros' 
          v-on:click='filtrar($event)'>
          Carros
        </button>
        
      </div> 


    <div class='container border rounded text-primary mt-4'>
      <div class='row lead border rounded font-weight-bold'>
        <div class='col-4' style='text-align:center'>
          Nombre del gasto
        </div>
        <div class='col-2'>
          Monto del gasto
        </div>
        <div class='col-3'>
          Tipo de gasto
        </div>
        <div class='col-3' style='text-align:center'>
          Accion
        </div>
      </div>
      <gastosComponente 
        v-for="(gastos,index) in gastos"
        v-bind:gastos='gastos'
        v-bind:id='gastos.id'
        v-bind:indice='index'
        v-bind:key='index'
        v-on:eliminarGasto='eliminar($event)'
        v-on:modificarGasto='modificar($event)' ></gastosComponente>
        <div class='row lead border rounded font-weight-bold'>
        <div class='col-3'>Valor Total</div>
        <div class='col-3'>{{valorTotal}}</div>
      </div>
    </div>
  </div>
  <div v-else>
    <loginForm v-bind:firebase="firebase"
               v-on:ingresoCorrecto='ingresoCorrecto($event)'></loginForm>
  </div>

</template>

<script>

import firebase from 'firebase'
import 'firebase/firestore'
import gastosComponente from './components/GastosComponente.vue'
import loginForm from './components/loginForm.vue'

export default {
  name: 'app',
  data: function() {
    return {gastos : [],
    b_hogar:'btn btn-default',
    b_trabajo:'btn btn-default',
    b_carros:'btn btn-default',
    b_todos:'btn btn-primary',
    inputNombre:'',
    inputMonto:'',
    inputTipo:'',
    coleccion:{},
    logon: false,
    firebase:'',
    idUsuario:'',
    db:'',
    valorTotal:0
    }
  },

  components: {
    gastosComponente, 
    loginForm
  },
  beforeMount: function () {
    var config = 
    {
    apiKey: "AIzaSyAo3W58HKX0FujA9QSpj8c1YMgZgz0koJo",
    authDomain: "libreta-de-gastos-b76a5.firebaseapp.com",
    databaseURL: "https://libreta-de-gastos-b76a5-default-rtdb.firebaseio.com",
    projectId: "libreta-de-gastos-b76a5",
    storageBucket: "libreta-de-gastos-b76a5.appspot.com",
    messagingSenderId: "256192724233"
   
  };

    firebase.initializeApp(config);
    this.db = firebase.firestore();
    const settings = {timestampsInSnapshots: true};
    this.db.settings(settings);
    this.firebase = firebase;
  }, 

  methods: {
    manejarClick: function (evento) {
      if (evento.target.id==='agregar'){
        const gastoData = {nombre:this.inputNombre,
                          monto:this.inputMonto,
                          tipo:this.inputTipo,
                          };
        this.coleccion.add(gastoData)
        .then((docReference) => {
          this.gastos.unshift({id:docReference.id, nombre: gastoData.nombre, monto: gastoData.monto, tipo: gastoData.tipo,
          });
        })
        .catch((Errro) => {
          alert('No se pudo agregar el gasto al sistema. Error: '+Errro.message);
        });
        this.inputNombre='';
        this.inputMonto='';
        this.inputTipo='';

          this.mostrarTodos();

      }else if(evento.target.id==='modificar'){
        const gastoData=({nombre:this.inputNombre,
                          monto:this.inputMonto,
                          tipo:this.inputTipo,
                          });
        this.coleccion.doc(this.gastosRef.id).set(gastoData)
        .then( () => {
          this.listaGastos.splice(this.gastosRef.indice,1,{id:this.gastosRef.id, nombre: gastoData.nombre, monto: gastoData.monto, tipo: gastoData.tipo,
          });
          this.btn='agregar';
          this.idBtn='agregar';
          this.calcular();
        })
        .catch((Errro) => {
          alert('No se pudo actualizar el gasto al sistema. Error: '+Errro.message);
        });
        this.inputNombre='';
        this.inputMonto='';
        this.inputTipo='';                 
      }
    },
    eliminar: function (GastosID){
      this.coleccion.doc(GastosID.id).delete();
      this.gastos.splice(GastosID.indice,1);
      this.valorTotal = Number(this.valorTotal) + Number(GastosID.monto);
    },
    modificar:function(GastosID){
       this.inputNombre=this.gastos[GastosID.indice].nombre;
       this.inputMonto=this.gastos[GastosID.indice].monto;
       this.inputTipo=this.gastos[GastosID.indice].tipo;
       this.btn='modificar';
       this.idBtn='modificar';
       this.gastosRef=GastosID;  
    },
    ingresoCorrecto: function(usuario) {
        console.log('User: '+usuario);
        this.idUsuario=usuario;
        this.logon=true;
        this.valorTotal=0;
        this.coleccion = this.db.collection('/usuarios/'+usuario+'/gastos');
        this.coleccion.get()
        .then((gastos)=>{
          gastos.forEach((gastos) => {
            this.gastos.push({
              id:gastos.id,
              nombre:gastos.data().nombre,
              monto:gastos.data().monto,
              tipo:gastos.data().tipo,
              })
               this.valorTotal = Number(this.valorTotal) + Number(gastos.data().monto);
        });
      });
    },
    filtrar: function (evento) {
    if (evento.target.id==='hogar'){
      this.hogar='btn btn-primary';
      this.trabajo='btn btn-default';
      this.carros='btn btn-default';
      this.todos='btn btn-default';
      this.valorTotal=0;
      this.gastos = [];
      let docRef = this.coleccion;
      let query = docRef.where("tipo","==","Hogar")
      query.get()
        .then((gastos)=>{
          gastos.forEach((gasto) => {
            this.gastos.push({
              id:gasto.id,
              nombre:gasto.data().nombre,
              monto:gasto.data().monto,
              tipo:gasto.data().tipo,
              });
              this.valorTotal = Number(this.valorTotal) + Number(gasto.data().monto);
            });
          });
    }
    else if (evento.target.id==='trabajo'){
      this.hogar='btn btn-default';
      this.trabajo='btn btn-primary';
      this.carros='btn btn-default';
      this.todos='btn btn-default';
      this.valorTotal=0;
      this.gastos = [];
      let docRef = this.colecicon;
      let query = docRef.where("tipo","==","Trabajo")
      query.get()
        .then((gastos)=>{
          gastos.forEach((gasto) => {
            this.gastos.push({
              id:gasto.id,
              nombre:gasto.data().nombre,
              monto:gasto.data().monto,
              tipo:gasto.data().tipo,
              });
              this.valorTotal = Number(this.valorTotal) + Number(gasto.data().monto);
            });
          });
    }
    else if (evento.target.id==='carros'){
      this.hogar='btn btn-default';
      this.trabajo='btn btn-default';
      this.carros='btn btn-primary';
      this.todos='btn btn-default';
      this.valorTotal=0;
      this.gastos = [];
      let docRef = this.coleccion;
      let query = docRef.where("tipo","==","Carros")
      query.get()
        .then((gastos)=>{
          gastos.forEach((gasto) => {
            this.gastos.push({
              id:gasto.id,
              nombre:gasto.data().nombre,
              monto:gasto.data().monto,
              tipo:gasto.data().tipo,
              });
              this.valorTotal = Number(this.valorTotal) + Number(gasto.data().monto);
            });
          });
    }
    else if (evento.target.id==='todos'){
      this.mostrarTodos();
    }
  },
   mostrarTodos: function (){
      this.hogar='btn btn-default';
      this.trabajo='btn btn-default';
      this.carros='btn btn-default';
      this.todos='btn btn-primary';
      this.valorTotal=0;
      this.gastos = [];
      let docRef = this.coleccion;
      docRef.get()
      .then((gastos)=>{
        gastos.forEach((gasto) => {
          this.gastos.push({
            id:gasto.id,
            nombre:gasto.data().nombre,
            monto:gasto.data().monto,
            tipo:gasto.data().tipo,
            });
            this.valorTotal = Number(this.valorTotal) + Number(gasto.data().monto);
          });

        });
     },
  },
  
  
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

<template>
  <div class='row lead my-1'>
    <div class='col-4'>{{gastos.nombre}}</div>
    <div class='col-2'>{{gastos.monto}}</div>
    <div class='col-3'>{{gastos.tipo}}</div>
    <div class='col-3'>
      <div id='modificar'  class='btn btn-primary'
           v-on:click='manejarClick($event,{indice:indice, id:id})'><i class="fa fa-pencil" aria-hidden="true"></i></div>
      <div id='eliminar'  class='btn btn-primary'
           v-on:click='manejarClick($event,{indice:indice, id:id})'><i class="fa fa-trash" aria-hidden="true"></i></div>     
    </div>
  </div>
</template>

<script>
export default {
  name: 'gastos',
  props: ['gastos','id','indice'],
  data: function () {
    return {
      test: 'test'
    }
  },
  methods: {
    manejarClick: function (evento,GastosID) {
      if (evento.target.id==='eliminar'){
        this.$emit('eliminarGasto',GastosID)
      }else if(evento.target.id==='modificar'){
        this.$emit('modificarGasto',GastosID)
      }
    }
  }
}
</script>

<style>

</style>

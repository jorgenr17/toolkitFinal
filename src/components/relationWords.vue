<template>
	<v-container fluid grid-list-sm>
    <v-content>
		<v-dialog v-model="addWord" max-width="500px" persistent>
			<v-card>
				<v-card-title class="headline">Agregar Categoria</v-card-title>
				<v-card-text>
					<v-text-field @keyup.enter="addWordInArray()" label="Palabra a agregar" v-model="word"
					placeholder color="accent" :persistent-hint="true"></v-text-field>
				</v-card-text>
				<v-card-actions>
					<v-btn color="primary" flat @click="closeDialog()">Cerrar</v-btn>
				</v-card-actions>
			</v-card>
		</v-dialog>

    <div>
      <v-alert :value="true" icon="info" color="primary" outline dismissible class="mt-5">
        Para construir las relaciones, asegurate de seleccionar la palabra clave que desees relacionar,
        luego agrega las categorias que desees presionando el boton agregar que se encuentra debajo de la caja de
        categorias. Para asociar las palabras relevantes a las categorias, presiona click en la categoria deseada
        y luego escoge las palabras que consideras asociar a esa categoria, el sistema te indicara cuales asociaste
        mediante un check.
      </v-alert>
    </div>
		<v-layout row wrap>
			<!-- <pre>{{this.$store.state.app.model}}</pre> -->
			<v-flex d-flex xs12 sm12 md4 class="pa-1">
				<v-card height="300px" class="elevation-7">
					<v-toolbar card color="black" class="white--text">
						<v-flex class="text-xs-center">
							<v-toolbar-title style="font-size: 28px">Palabras Claves</v-toolbar-title>
						</v-flex>
					</v-toolbar>
					<v-card-text class="list">
						<v-list dense>
							<v-list-tile v-for="(value, index) in topics" :key="index" :class="{activo: value.active }" @click="showChild(index), child = index, category = ''">{{value.palabraClave}}
								<v-spacer></v-spacer>
								<v-avatar class="grey white--text font-weight-bold" size="25px">{{value.temasDeInteres.length}}</v-avatar>
								<v-icon>keyboard_arrow_right</v-icon>
							</v-list-tile>
						</v-list>
					</v-card-text>
				</v-card>
			</v-flex>

			<v-flex d-flex xs12 sm12 md4 class="pa-1">
				<v-card height="300px" class="elevation-7">
					<v-toolbar card color="black" class="white--text">
						<v-flex class="text-xs-center">
							<v-toolbar-title style="font-size: 28px">Categorías</v-toolbar-title>
						</v-flex>
					</v-toolbar>
					<!-- <div class="pr-1"> -->
					<v-btn small absolute dark fab right bottom color="info" @click="addWord = !addWord"><v-icon>add</v-icon></v-btn>
					<!-- </div> -->
					<v-card-text class="list" v-if="child !== '' && this.$store.state.app.model.relaciones[this.child].temasDeInteres.length !== 0">
						<v-chip v-for="(value, index) in this.$store.state.app.model.relaciones[this.child].temasDeInteres" :key="index"
            color="grey" dark small @click="relation(index)" close @input="deleteWord(index)">
							<span class="pr-2" style="font-size: 20px; font-weight: bold">#</span>{{value.temaDeInteres}}
						</v-chip>
					</v-card-text>
					<v-card-text v-else>
						<div style="font-size: 16px; color: grey; text-align: center">No hay categorías para mostrar. Deseas agregar alguna?</div>
					</v-card-text>
				</v-card>
			</v-flex>

			<v-flex d-flex xs12 sm12 md4 class="pa-1">
				<v-card height="300px" class="elevation-7">
					<v-toolbar card color="black" class="white--text">
						<v-flex class="text-xs-center">
							<v-toolbar-title style="font-size: 28px">Palabras Relevantes</v-toolbar-title>
						</v-flex>
					</v-toolbar>
					<v-card-text class="list">
						<div v-if="category === ''" style="font-size: 16px; color: grey; text-align: center">Selecciona las palabras relevantes a relacionar con la categoria</div>
						<div v-else>
							<v-scroll-x-transition group hide-on-leave>
								<v-chip color="grey" dark small v-for="(value, i) in commonElements" :key="i" @click="asignWord(value.word)">{{value.word}}<v-icon v-if="value.val">check_circle</v-icon>
								</v-chip>
							</v-scroll-x-transition>
						</div>
					</v-card-text>
				</v-card>
			</v-flex>
      <pre v-if="child !== ''">{{this.$store.state.app.model.relaciones[this.child]}}</pre>
		</v-layout>
		<!-- <div class="text-xs-center pa-5">
			<v-btn @click="sendWords()" color="black" class="accent--text">Enviar Relaciones</v-btn>
		</div> -->
    <div class="text-xs-justify pt-4">
      <v-btn color="black" left absolute outline to="/UsingIA/palabraClave">Atrás</v-btn>
      <v-btn color="black" right absolute outline @click="next('/UsingIA/questions')">Siguiente</v-btn>
    </div>
  </v-content>
	</v-container>
</template>

<script>
import EventBus from '@/components/EventBus'
import { url } from '../main'

export default {
  name: 'relationWords',
  data: () => ({
    child: '',
    child2: '',
    addWord: false,
    word: '',
    activo: false,
    category: '',
    category2: ''
    // url: 'https://carinag-225014.appspot.com'
  }),
  methods: {
    next (to) {
      let el = this.$store.state.app.application.usingCM.steps['E'].data.find(el => el.to === to)
      this.$store.commit('app/changeElements', {to: to, el: el, i: 'E'})
      this.$store.commit('app/changeStep', 'E')
      EventBus.$emit('init', 'E')
    },
    showChild (index) {
      this.child = index
    },
    deleteWord (index) {
      this.$store.commit('app/deleteCategory', { i: index, c: this.child })
      // this.child = ''
      this.category = ''
    },
    addWordInArray () {
      if (this.child !== '') {
        if (this.word !== '') {
          this.$store.commit('app/addCategory', { word: this.word, child: this.child })
          this.word = ''
        } else {
          alert('Por favor ingresa una palabra.')
        }
      } else {
        alert('Por favor selecciona un tema.')
      }
    },
    closeDialog () {
      this.addWord = false
      this.word = ''
    },
    relation (index) {
      this.category = index
    },
    asignWord (word) {
      if (this.category !== '') {
        this.$store.commit('app/asignWord', { word: word, child: this.child, category: this.category })
      } else {
        alert('Por favor selecciona una categoria.')
      }
    },
    sendWords () {
      EventBus.$emit('loading', {loading: true})
      this.$store.dispatch('app/relationedWords', url)
    },
    active (index) {
      if (this.child === index) {
        this.activo = true
      } else {
        this.activo = false
      }
    }
  },
  computed: {
    commonElements () {
      let array = []
      this.$store.state.app.model.contexto.palabrasRelevantes.map(word => {
        let index = this.$store.state.app.model.relaciones[this.child].temasDeInteres[this.category].palabrasRelevantes.indexOf(word)
        if (index !== -1) {
          array.push({ word: word, val: true })
        } else {
          array.push({ word: word, val: false })
        }
      })
      return array
    },
    topics () {
      let array = []
      this.$store.state.app.model.relaciones.map(el => {
        if (this.$store.state.app.model.relaciones.indexOf(el) === this.child) {
          el.active = true
        } else {
          el.active = false
        }
        array.push(el)
      })
      return array
    },
    objReturn () {
      let a = this.$store.state.app.model.relaciones[this.child]
      return a[this.category]
    }
  },
  components: { EventBus }
}
</script>

<style lang="css">
.activo {
	background-color: rgb(53, 204, 204, 0.2);
	border-radius: 15px
}
.activo:hover {
	background-color: rgb(53, 204, 204, 0.2);
	border-radius: 15px
}
.list{min-height:78.5%; max-height:78.5%; width:100%; list-style:none;}
.list{overflow:hidden; overflow-y:scroll;}
</style>

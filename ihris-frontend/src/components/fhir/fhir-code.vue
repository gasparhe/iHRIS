<template>
  <ihris-element :edit="edit" :loading="loading">
    <template #form>
      <v-select 
        :loading="loading" 
        :label="display" 
        v-model="value" 
        :items="items" 
        outlined 
        hide-details="auto" 
        :error-messages="err_messages"
        :error="error"
        item-text="display"
        item-value="code"
        :disabled="disabled"
        dense
      ></v-select>
    </template>
    <template #header>
      {{display}}
    </template>
    <template #value>
      {{displayValue}}
    </template>
  </ihris-element>
</template>

<script>
import IhrisElement from "../ihris/ihris-element.vue"

/*
const itemSort = (a,b) => {
  return (a.display === b.display ? (a.code === b.code ? 0 : (a.code < b.code ? -1: 1)) : (a.display < b.display ? -1 : 1) )
}
*/
export default {
  name: "fhir-code",
  props: ["field","min","max","base-min","base-max","label","binding","slotProps","path","edit","sliceName","readOnlyIfSet"],
  components: {
    IhrisElement
  },
  data: function() {
    return {
      value: "",
      loading: true,
      err_messages: null,
      error: false,
      items: [],
      source: { path: "", data: {}, binding: this.binding },
      disabled: false
    }
  },
  created: function() {
    this.setupData()
  },
  watch: {
    slotProps: {
      handler() {
        //console.log("WATCH CODE",this.field,this.path,this.slotProps)
        this.setupData()
      },
      deep: true
    }
  },
  methods: {
    setupData() {
      if ( this.slotProps && this.slotProps.source ) {
        this.source = { path: this.slotProps.source.path+"."+this.field, data: {} }
        if ( this.slotProps.source.fromArray ) {
          this.source.data = this.slotProps.source.data
          this.value = this.source.data
          //console.log("SET value to ", this.source.data, this.slotProps.input)
        } else {
          let expression = this.$fhirutils.pathFieldExpression( this.field )
          this.source.data = this.$fhirpath.evaluate( this.slotProps.source.data, expression )
          //console.log("STR FHIRPATH", this.slotProps.source.data, this.field)
          if ( this.source.data.length == 1 ) {
            this.value = this.source.data[0]
          }
        }
        this.disabled = this.readOnlyIfSet && (!!this.value)
        //console.log(this.source)
      }
      let binding = this.binding || this.slotProps.source.binding
      this.$fhirutils.expand( binding ).then( items => {
        this.items = items 
        this.loading = false
      } ).catch( err => {
        console.log(err)
        this.error = true
        this.err_messages = err.message
        this.loading = false
      } )
      /*
      let lastSlash = binding.lastIndexOf('/')
      let lastPipe = binding.lastIndexOf('|')
      let valueSetId = binding.slice(lastSlash+1, (lastPipe !== -1 ? lastPipe : binding.length ))
      fetch("/fhir/ValueSet/"+valueSetId+"/$expand").then(response=> {
        if( response.ok ) {
          response.json().then(data=>{
            this.loading = false
            try {
              this.items = data.expansion.contains
              this.items.sort( itemSort )
            } catch(err) {
              this.error = true
              this.err_messages = "Invalid response from server."
            }
          }).catch(err=>{
            this.err_messages = err.message
            this.error = true
            this.loading = false
          })
        } else {
          // Try loading valueset without expansion if expand failed.
          console.log("Failed to get ValueSet Expansion for "+valueSetId)
          fetch("/fhir/ValueSet/"+valueSetId).then(response=> {
            if ( response.ok ) {
              response.json().then(data=> {
                this.items = []
                if ( data.compose.include ) {
                  for( let include of data.compose.include ) {
                    if ( include.concept ) {
                      for ( let concept of include.concept ) {
                        //this.items.push( { system: include.system, ...concept } )
                        concept.system = include.system
                        this.items.push( concept )
                      }
                    }
                  }
                }
                this.items.sort( itemSort )
                this.loading = false
              }).catch(err=>{
                this.err_messages = err.message
                this.error = true
                this.loading = false
              })
            } else {
              this.error = true
              this.err_messages = "Invalid response from server."
              this.loading = false
            }
          }).catch(err=>{
            this.err_messages = err.message
            this.error = true
            this.loading = false
          })

        }
      }).catch(err=>{
        this.err_messages = err.message
        this.error = true
        this.loading = false
      })
      */
    }
  },
  computed: {
    display: function() {
      if ( this.slotProps && this.slotProps.input) return this.slotProps.input.label
      else return this.label
    },
    displayValue: function() {
      let found = this.items.find( item => item.code === this.value )
      if ( found ) {
        return found.display
      } else {
        return ""
      }
    }
  }
}
</script>

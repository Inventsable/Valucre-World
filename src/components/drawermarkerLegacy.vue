<template>
    <div>
        <v-list-group 
            v-for="type in markertypes" 
            v-show="type.show"
            :key="type.name" 
            
            :value="type.value">
            <template v-slot:activator>
                <v-list-tile>
                    <v-list-tile-title class="text-uppercase typeTitle">{{type.title}}</v-list-tile-title>
                </v-list-tile>
            </template>
              <v-expansion-panel focusable v-model="type.foci">
                 <v-expansion-panel-content v-for="(item,i) in type.markers" :key="i">
                     <template v-slot:header>
                         <div class="markerTitle">{{item.title}}</div>
                     </template>
                     <div class="prependMarkerLink" v-show="hasPermissions">
                         <v-btn icon>
                             <v-icon color="grey">edit_location</v-icon>
                         </v-btn>
                     </div>
                     <div class="appendMarker2Link">
                         <v-btn icon>
                             <v-icon color="grey">link</v-icon>
                         </v-btn>
                     </div>
                     <v-card>
                         <v-card-text
                             class="grey lighten-3"
                         >{{item.desc}}</v-card-text>
                     </v-card>
                 </v-expansion-panel-content>
              </v-expansion-panel>
        </v-list-group>
    </div>
</template>

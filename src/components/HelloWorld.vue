<template>
  <v-container>
    <div>     
      <v-row>
        <!-- Setting section -->
        <v-col cols="4" >
       <v-select
            :items=[5,10,15,20,25,30]
            v-model = "recPerPage"
            label="Records per page"
            @change="reloadItem('root')"
          ></v-select>
        </v-col>
        <v-col cols="4" >
          <v-select
            :items=orderByList
            v-model = "orderByField"
            label="Order by field"
            @change="reloadItem('root')"
          ></v-select> 
        </v-col>
        <v-col cols="4" >
          <v-select
            :items=orderDirectionList
            v-model = "orderDirection"
            label="Order direction"
            @change="reloadItem('root')"
          ></v-select>
        </v-col>
      </v-row>
        </div>
     <v-treeview v-if="!dialog"
          :items="itemsw"
          item-key="Id"
          :open.sync="open"
          item-text = "content"
          item-children ="rows" 
          hoverable  
          :load-children="fetchNode" 
          treeview-node-level-width = "180px"      
          >

      <template v-slot:label ="{ item, open }" elevation="4">
          <v-card  elevation="4" v-if="item.Id=='root'">
            <v-row>
              <v-col cols="12" align="right">
                <v-btn color="teal accent-3"  @click="btnClick(item)">start a new topic</v-btn>
              </v-col>
            </v-row>
          </v-card>
        
          <v-card  elevation="4" v-if="item.Id!=='root'">
           <v-alert
              border="left" colored-border  elevation="4"
              color="deep-purple accent-4"
              >
              <v-row >
                <v-col cols="10" > 
                  <b>
                  {{ localeDateTime(item.createdAt) }} 
                  {{ item.userName }}  
                  {{item.email}}
                  <span v-if="item.url">url:{{item.url}}</span></b>
                  <br><br>
                  <span  v-html=item.content> </span>
                </v-col>

                <v-col cols="1" > 
                  <div v-if="item.originalFile">
                      <v-img   max-width="80" :src=getFileUrl(item) 
                          @click="downloadImg(item.Id)" 
                          v-if="item.mimeFile!='text/plain'"></v-img>
                      <v-icon x-large color="teal" @click="downloadTxt(item.Id)" 
                            v-if="item.mimeFile==='text/plain'"> mdi-text-long 
                      </v-icon>
                  </div>        
                </v-col>

                <v-col cols="1" align="right">
                    <v-icon x-large color="teal"  @click="btnClick(item)"> mdi-reply </v-icon>
                </v-col>
              </v-row>
            </v-alert>

          </v-card>

          <div class="text-center">
              <br  v-if="item.Id=='root'">
              <v-pagination
                  v-if="item.count>recPerPage"
                  v-model="item.page"
                  :length="(Math.ceil(item.count/recPerPage))"
                  circle dense
                  @input="reloadItem(item.Id, item.page)"
              ></v-pagination>
          </div>
      </template>
    </v-treeview>

    <v-card  elevation="4" v-if="dialog">
      <v-banner  elevation="5">
        <v-row >
          <v-col cols="10" > 
            <b>
            {{ localeDateTime(activeItem.createdAt) }} 
            {{ activeItem.userName }}  
            {{activeItem.email}}
            <span v-if="activeItem.url">url:{{activeItem.url}}</span></b>
            <br><br>
            <span  v-html=activeItem.content> </span>
          </v-col>
          <v-col cols="2" v-if="activeItem.originalFile"> 
              <v-img   max-width="80" :src=getFileUrl(activeItem) 
                      @click="downloadImg(activeItem.Id)"
                      v-if="activeItem.mimeFile!='text/plain'">
              </v-img>
      
              <v-icon x-large color="teal" @click="downloadTxt(activeItem.Id)" 
                    v-if="activeItem.mimeFile==='text/plain'"> mdi-text-long 
              </v-icon>
          </v-col>
        </v-row>
      </v-banner>
      
      <v-banner color="gray">
        <br>
        <form id="formElem">
        <v-row >
          <v-col cols="3" >
            <v-text-field
                name = "userName"
                v-model="newItem.userName"
                :rules="[rules.required, rules.validName]"
                label="User name" dense outlined 
            ></v-text-field>
          </v-col>
          <v-col cols="3" >
            <v-text-field
                name = "email"
                v-model="newItem.email"
                :rules="[rules.required, rules.email]"
                label="email" dense outlined 
            ></v-text-field>
          </v-col>
          <v-col cols="2" >
            <v-text-field
                name = "homePage"
                v-model="newItem.homePage" dense outlined
                label="Home Page"
            ></v-text-field>
          </v-col>
          <v-col cols="3" >
            <v-file-input show-size label="File" outlined dense clearable single-line
                name = "attFile"
                :rules="[rules.validfile]"   accept="image/png, image/jpeg, image/gif, text/plain" 
                @change="Preview_image"
                @click:clear="url=null"
                v-model="image"   > 
            </v-file-input>
          </v-col>
          <v-col cols="1" >
            <v-img   max-width="80" :src="url"></v-img>
          </v-col>
        </v-row>

        <v-row >
          <v-col cols="10"> 
            <v-textarea
                counter ref="textarea"
                outlined
                label="Your comment:"
                v-model="newItem.content"
                :rules="[rules.required, rules.validtags]"
            ></v-textarea>
          </v-col>
          <v-col cols="2">  
            <v-btn small @click='execInsertText(`<a href="" title=""></a>`)'>a href</v-btn>
            <v-btn small @click="execInsertText(`<code></code>`)">code</v-btn>
            <v-btn small @click="execInsertText(`<strong></strong>`)">strong</v-btn>
            <v-btn small @click="execInsertText(`<i></i>`)">italic</v-btn>
          </v-col>
        </v-row>

        <v-row >
          <v-col cols="9">
            <span  v-html=previewComment(newItem.content)> </span>
          </v-col>
          <v-col cols="3">
            <v-btn color="error" @click="dialog=false">Cancel</v-btn>
            <v-btn color="primary" @click="submitPost">Submit</v-btn>
          </v-col>
        </v-row>
        </form>
      </v-banner>
    </v-card>

    <v-dialog v-model="showImg" width="500">
        <v-card>
          <v-card-text>
             <v-img  :src=previewImageUrl ></v-img>
          </v-card-text>
        </v-card>
    </v-dialog>

    <v-dialog v-model="showTxt" width="800" >
        <v-card>
          <v-card-text>
            <v-textarea v-model="previewTextF" outlined></v-textarea>
          </v-card-text>
        </v-card>
    </v-dialog>

    <v-snackbar v-model="snackbar" color="red" centered>
             Server rejected this message
        <template v-slot:action="{ attrs }">
          <v-btn @click="snackbar = false">Close</v-btn>
        </template>
    </v-snackbar>
    
  </v-container>
</template>

<script>
import conf from '../config.js';
const  {removeBannedTags, chkHtmlTags} = require('../commentcleaner.js');

export default {
    name: 'HelloWorld',
    data: () => ({
      recPerPage: 5,
      orderByField: 'createdAt',
      orderByList: ['userName', 'email', 'homePage', 'createdAt'],
      orderDirection: 'DESC',
      orderDirectionList:['ASC', 'DESC'],
      url: null,
      image: null,
      previewTextF:'',
      previewImageUrl:'',
      open: [],
      dialog: false,
      showImg: false,
      showTxt: false,
      snackbar: false,
      newItem: { userName:'', email:'',homePage:'', content: '' },
      activeItem: {Id: 'root', content: 'Comments :', rows: [] },
      items: [ {Id: 'root', content: 'Comments :', rows: [] } ],

      rules: {
        required: value => !!value || 'Required.',
       
        validName: value => {
          const pattern = /^[A-z0-9]{1,64}$/
          return pattern.test(value) || 'Invalid name.'
        },  

        email: value => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
          return pattern.test(value) || 'Invalid e-mail.'
        },

        validtags: value => {
          return chkHtmlTags(removeBannedTags(value)).length==0 || 'Tags error.'
        },

        validfile: value => {
          return !value ||  ["image/jpeg", "image/jpg", "image/png", "image/gif"].includes(value.type) || value.type == "text/plain" && value.size < 102400||  'Text file should be less than 100kB !'
        },
      },
        
    }),
    computed: {
      itemsw: function() { return this.items }
        },

    methods:{


      async reloadItem(item, page=1 ){
        
        var element = this.items[0];
        var result = this.searchTree(element, item);
          this.fetchNode( result, this.recPerPage, page-1 )
          this.open.push(item)
        },


      searchTree: function (element, matchingId){
        if(element.Id == matchingId){
            return element;
          }else if (element.rows != null){
            var i;
            var result = null;
            for(i=0; result == null && i < element.rows.length; i++){
               result = this.searchTree(element.rows[i], matchingId)}
            return result;
          }
        return null;
      },


      async submitPost(){

            var input = document.querySelector('input[type="file"]')
            var data = new FormData()
            data.append('attFile', input.files[0])
            data.append('userName', this.newItem.userName)
            data.append('email', this.newItem.email)
            if (this.newItem.homePage.length>0) 
                {data.append('homePage', this.newItem.homePage)}
            data.append('content', this.newItem.content)

            const resp = await fetch(conf.urlapi+'/'+this.activeItem.Id, {
                  method: 'POST',
                  body: data
            })
            if (resp.status==200) {
                  this.reloadItem(this.activeItem.Id)
                  this.dialog = false
              } else {
                  this.snackbar=true;
                }
      },

      downloadImg (Id){
          this.showImg = true;
          this.previewImageUrl = conf.urlapi+'/download/'+Id;    
      },

      async downloadTxt(Id){
          this.showTxt = true;
          let url = conf.urlapi+'/download/'+Id;
          this.previewTextF = await (await fetch(url)).text() 
      },

      getFileUrl(item) {
          if (item.originalFile){
            return conf.urlapi+'/download/'+item.Id
          }},

      Preview_image() {
        if (!!this.image){
            this.url= URL.createObjectURL(this.image) }
      },

      execInsertText(text) {
        
          const insertText = text
              if (!insertText.length) return
                  const textarea = this.$refs.textarea.$refs.input
      
          const sentence = textarea.value
          const len = sentence.length
          let pos = textarea.selectionStart
          if (pos === undefined) { pos = 0}
          if (pos==0 && this.newItem.content){ pos= this.newItem.content.length }
          const before = sentence.substr(0, pos)
          const after = sentence.substr(pos, len)
          this.newItem.content = before + insertText + after
      },

      previewComment( inputText) {
          return removeBannedTags(inputText);
      },

      localeDateTime(val) {
          const d = new Date(val);
          return  d.toLocaleDateString() +'  '+ String(d.getHours()).padStart(2, '0')+':' + String(d.getMinutes()).padStart(2, '0') 
      },
     
      btnClick (item, open) {
          this.dialog = !this.dialog;
          this.activeItem = item;
      },
    
      async fetchNode (item, count = this.recPerPage, page = 0, orderby = this.orderByField, direction = this.orderDirection) {

            const responseServ = await (await fetch(conf.urlapi+'/'+item.Id+
                  '/'+count+'/'+page+'/'+orderby+'/'+direction,
                  {method: 'GET', mode: 'cors', headers: {'Content-Type':'application/json'} })).json()
            
            item.count = await responseServ.count;
            let newrows =  await responseServ.rows;
            newrows.map(function(current) { 
                  current.rows = [];})
            item.rows = newrows 
      }
    },
  }
</script>

<template>
    <div>
    <el-upload
      drag
      action
      :auto-upload="false" 
      :show-file-list="false" 
      :before-upload="beforeUp"
      :on-change='change'
      
      >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传1111</em></div>
    </el-upload>
    <el-button style="margin-left: 10px;" size="small" type="success" @click="startUp(indexC)">上传到服务器</el-button>
    </div>
</template>
<script>
import axios from 'axios'
import SparkMD5 from "spark-md5";
export default {
    data(){
        return{
            bufferFile:'',
            md5:'',
            allChuck:'',
            allSlice:[],
            size:1024*1024,
            fileCopy:'',
            indexC:''
        }
    },
    methods:{
        beforeUp(file){
            console.log(file);
           
            if(file.size<=(1024*1024)){
                console.log(file);
                //axios.post('/api/')
                return true
            }else{
               this.allChuck=Math.ceil(file.size/(1024*1024));
               console.log(this.allChuck,7777)
               this.checkMd5(file);
            }
        },
        change(file) {
             this.allSlice=[]; 
             this.fileCopy=file.raw;
             this.beforeUp(this.fileCopy)
        },
      async checkMd5(file){
          var self=this;
          const fileRead = new FileReader();
          const sparkMD5 = new SparkMD5.ArrayBuffer()
          await fileRead.readAsArrayBuffer(file);
          await fileRead.addEventListener('load',function(e){
              console.log(e);
              self.bufferFile=e.target.result;
              for(let i=0;i<self.allChuck;i++){
                console.log(i,89);
                
               self.allSlice.push(new Blob([self.bufferFile.slice(i*self.size,(i+1)*self.size)]));
               
              }
              console.log(self.allSlice,9999)  
          });
          
          
          sparkMD5.append(this.bufferFile);
          this.md5= sparkMD5.end();
          console.log(this.md5);
          await this.checkUp(this.md5);
      },
      //上传md5
      checkUp(md5){
          var self=this;
          axios.post('/api/file/fragment/checkFile',{fileMd5:md5}).then(res=>{
              console.log(res);
              self.indexC=res.data.data.currIndex;
              //self.startUp(self.indexC)  
          })
      },
      //开始切片并且上传
       startUp(index){
           console.log(index)
           let current=index-1;
           var self=this;
           if(current<this.allSlice.length){
               console.log(111);
               const formData = new FormData();
               var fragmentDto={
                   type:'file',
                   moduleCode:'locus',
                   originFileName:self.fileCopy.name,
                   fileMd5:self.md5,
                   fragmentNum:this.allSlice.length,
                   currIndex:index,
                   size:self.fileCopy.size
               }
               formData.append('file',this.allSlice[current]);
               formData.append('fragmentDto',JSON.stringify(fragmentDto));
               axios.post('/api/file/fragment/upload',formData).then(res=>{
                    self.indexC++;
                    self.startUp(self.indexC)
               }) 
           }else{
               
               return
           }

       } 
    }
}
</script>

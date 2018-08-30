# 代码snippets

1. 实现bind

   ```js
   Function.prototype.bind = function() {
       var fn = this,
           args = Array.prototype.slice.call(arguments),
           object = args.shift();
       return function(){
           fn.apply(object, args.concat(Array.prototype.slice.call(arguments)));
       }
   }
   ```

2. 实现curry

   ```js
   Function.prototype.curry = function() {
       var fn = this,
           args = Array.prototype.slice.call(arguments);
       return function() {
           fn.apply(this, args.concat(Array.prototype.slice.call(arguments)))
       }
   }
   ```

3. 更复杂的curry

   ```js
   Function.prototype.partition = function(){
       var fn = this,
           args = Array.prototype.slice.call(arguments);
       return function() {
           var args1 = Array.prototype.slice.call(arguments);
           for(var i = 0; i < args.length; i++) {
               if(!args[i]) {
                   args[i] = args1.shift();
               }
           }
           fn.apply(this, args.concat(args1));
       }
   }
   ```

4. Promise封装的xhr

   ```js
   function ajax(url, type, options){
       return new Promise(function(resolve,reject) {
           var xhr;
           if(window.XMLHttpRequest) {
               xhr = new XMLHttpRequest();
           } else {
               xhr = new ActiveXObject('Microsoft.XMLHTTP');
           }
           xhr.onreadystatechange = function() {
               if(xhr.readyState == 4 ) {
                   var status = xhr.status;
                   if(status >= 200 && status <300) {
                       resolve(xhr.responseText);
                   } else {
                       reject(status);
                   }
               }
           }
           
           if(type == 'GET') {
               xhr.open('GET', url);
           } else {
               xhr.open('POST', url);
               xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
           }
           xhr.send();	
       })
   }
   ```

5. JSONP

   http://www.cnblogs.com/chopper/archive/2012/03/24/2403945.html

   ```js
   <script type="text/javascript">
       //添加<script>标签的方法
       function addScriptTag(src){
           var script = document.createElement('script');
           script.setAttribute("type","text/javascript");
           script.src = src;
           document.body.appendChild(script);
       }
       
       window.onload = function(){
           //搜索apple，将自定义的回调函数名result传入callback参数中
           addScriptTag("http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=apple&callback=result");
           
       }
       //自定义的回调函数result
       function result(data) {
           //我们就简单的获取apple搜索结果的第一条记录中url数据
           alert(data.responseData.results[0].unescapedUrl);
       }
   </script>
   ```

6. Debounce

   ```js
   var debounce = function(action, delay) {
       var timer = null;
       return function() {
           var self = this, args = arguments;
           timer&&clearTimeout(timer);
           timer = setTimeout(function(){
               action.apply(self,arguments);
           },delay);
       }
   }
   ```

7. Throttle 

   ```js
   var throttle = function(action, delay) {
       var startTime = 0;
       var timer;
       return function() {
           var self = this, args = arguments;
           var currentTime = new Date();
           if(currentTime - startTime >= delay) {
               action.apply(this, arguments);
               startTime = currentTime;
           } else {
               timer&&clearTimeout(timer);
               timer = setTimeout(function(){
                   startTime = currentTime;
                   action.apply(self, args);
               }, delay)
           }
       }
   }
   ```

8. 深拷贝

   ```js
   function deepCopy(source, target = {}) {
       var key;
       for(key in source) {
           if(source.hasOwnProperty[key]) {
               if(typeof(source[key]) === 'object') {
                   target[key] = Array.isArray(source[key]) ? [] : {};
                   deepCopy(source[key], target[key]);
               } else {
                   target[key] = source[key];
               }
           }
       }
       return target;
   }
   ```

   解决环的问题

   https://www.imooc.com/article/39933

   ```js
   function isObj(obj) {    
       return (typeof obj === 'object' || typeof obj === 'function') && obj !== null
   }
   function deepCopy(obj, hash = new WeakMap()) { 
       if(hash.has(obj)) return hash.get(obj)    
       let cloneObj = Array.isArray(obj) ? [] : {}     
       hash.set(obj, cloneObj)    
       for (let key in obj) {         
           cloneObj[key] = isObj(obj[key]) ? deepCopy(obj[key], hash) : obj[key];     
       }   
       return cloneObj
   } 
   ```

   
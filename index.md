<html>
<head>
    <style>
        *   {
            font-family: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif;

        }
        .task   {
            display:inline-block;
            position: relative;
            margin: 5% 5% 0 5%;
            padding: 5%;
            border-radius: 4px;
            box-shadow: 5px 5px 5px #ddd;
            background-color: white;
            width:80%;
            height:fit-content;
            overflow: hidden;
        }
        .prereqs {
            margin-left: 20px;
            font-size:small;
        }
        #ToDo   {
            height: 90%;
            overflow:scroll;
            background-color:#f8f8f8;
        }
        .cando  {color:black;}
        .cantbedone {
            color:lightgray;
            max-height: fit-content;
            height:max-content;
            display: inline-block;
            display: -none;        
        }
        .filtered {
            display:none;
}
        span.dupButton {
            position: absolute;
            top:0;right:0;
            font-weight:90;
            color:white;
            background-color:rgb(206, 199, 199);
            overflow:hidden;
            }
        span.dupButton:hover {
            overflow:visible;
        }
        .buttonBar {
            display:flex;
            bottom:2em;
            justify-content: space-evenly;
            background-color: blue;
            padding: .25em;

        }
        .button {
            color:white;
            background-color: darkblue;
            padding: .75em;
            border-radius: .5em;
            border-color: green;
        }

    </style>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta http-equiv="Content-Type" content="text/html; charset=windows-1252">
    <script>
        /* build data structure if it doesnt exist
        var state = {
            taskname : ["@walmart","prunes","kimchi for prunes","@truck", // the task description 
                        "install charger", "replace airbag fuse","disconnect battery","install plugstrip",
                        "eggs","tostados","creamer",
                        "@to-do-app",
                            "implement filters","style as cards","animate css","allow edits",
                            "design prereq inputs","auto-archive completed","implement drag and drop","","",
                    "" ],
            order : [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20], // a sorted array of indexes
            done : [0,0,20000+Date.now(),0,0,0], // false or past or future datetime
            prereqs : [ [],[0],[0,1],[],[3],[3,6],[3],[3],[0],[0],
                        [0],[],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11],[11] ] // an array of arrays of prerequisite ids 
        }
      */
        var state;   // create global state
        function renderAll(){
            getState();
            document.querySelector("div#ToDo").innerHTML="";
        
            state.order.map((id,index)=>renderTask(id,index));
            filterBy(state.filter);

        }
        function renderButtons(){
            return `
            <i class=""

            `
        }
        function setFilter(filter){
            state.filter=filter;
            setState();
            renderAll();
        }
        function filterBy(filter){ // Called from renderTask
            // adds the .filtered class to all div.task that should be hidden
            document.querySelectorAll(".filtered").forEach(task=>task.classList.remove('filtered'))
            let filteredTasks = 
                    document.querySelectorAll(filter);
                    // console.log(filteredTasks)
                    filteredTasks.forEach((task) => task.classList.add("filtered"));
        }
        function dupNewTask(id,index){
            console.log("dupNewTask",id,index)
             newTaskId=state.order.length;
            state.taskname[newTaskId]="new task";
            state.done[newTaskId]=false;
            state.prereqs[newTaskId] = state.prereqs[state.order[id]];
            state.order.splice(index,0,newTaskId)
            setState();
            renderAll();
        }
        function deleteTask(id){
            let position = state.order.indexOf(id)
            console.log(position);
            console.log(state.order.splice(position,1))

        }
        function dragHandler(event){
            if(event.offsetY<5) moveUp(event.target.id);
        }
        function renderPrereq(item){
            return '<div class="prereqs">' + state.taskname[item] + '</div>'
        }
        function renderPrereqs(id){
            let prereqshtml ='';
            prereqshtml = state.prereqs[id].map(renderPrereq).join('');
            return prereqshtml
        }
        function renderTask(id,index){
            const toDoList=document.querySelector("#toDo")
            const task=document.createElement('div');

            task.classList.add('task');

            if(canBeDone(id)) {task.classList.add('canBeDone')}
            else {task.classList.add('cantbedone')}

            if(isDone(id)) task.classList.add('isDone')
            else {task.classList.add('notDone')};
            
            task.setAttribute("draggable","true");
            task.setAttribute("id",index);
            task.addEventListener("dragover",e=>dragHandler(e))

            let checkBoxText='\u2612';
            
            if(canBeDone(id)) checkBoxText = '\u2610';
            if(isDone(id)) checkBoxText='\u2611';
            
            task.innerHTML = '<span class="doneBox" onclick="updateDone('+ id +')">'+ 
                                checkBoxText + '</span>' +
                                '<span class="taskname" id=' + id + ' draggable="true" contenteditable onblur="updateText(event.target.innerText,id)">'+ state.taskname[id] +'</span>' +
                                '<span class="dupButton" id='+ id +' onclick="dupNewTask(' + id + ',' + index + ')">\u29c9<br>\u270e<br>\u232b<br>\u2191<br>\u2193</span>' +
                                renderPrereqs(id); 
            
            toDoList.appendChild(task)

        
        }
        function setState(){
            localStorage.setItem("state", JSON.stringify(state))
        }
        function getState(){
            state = JSON.parse(localStorage.getItem("state"));
        }
/*        function renderTask2(id){
            
            const toDoList=document.querySelector("#ToDo");
            const task=document.createElement('div');
            const dupButton = document.createElement('span');

            task.setAttribute('contenteditable',true)
            task.innerHTML = "<span class=checkbox>\u2610</span>"
            task.innerHTML += state.taskname[id];
            task.classList.add('card');
            console.log(id);
            task.setAttribute("data-id",id);

            dupButton.textContent='+'
            dupButton.className='dup';
            dupButton.id=id;

            task.appendChild(dupButton)
            toDoList.appendChild(task)
        }
        */
        function isDone(id){
            if(!state.done[id]) return false;
            if(state.done[id]>Date.now()) return false;
            return true;
        }
        function canBeDone(id){
            for (const i of state.prereqs[id]) {
                if(!isDone(i)) return false;
            }
            return true;
            //      for each item in prereq[textid]
            //          if done[prereq[textid][item]]==false
            //              return item cant be done
            //           else next item
           //             next
        }
        // contenteditable callbacks
        function updateText(value,id){
            state.taskname[id]=value;
            setState();

        }
        function moveUp(id){
            if(id<=1) return;
            let swap = state.order[id-1];
            state.order[id-1] = state.order[id];
            state.order[id] = swap;
            setState();
            renderAll();
        }
        function moveDown(id){}
        function updateDone(id){
            if(!isDone(id) && canBeDone(id)) { state.done[id]=Date.now();}
            else {state.done[id]=false}
            setState();
            renderAll();
        }
        function deleteItem(id){}
        function updatePrereqs(id){}
    </script>
 
<!---
    ToDo:
 
-->
</head>
<body>
    <div id="ToDo"></div>
    <div class="buttonBar">
        <button class="canBeDoneButton button" onclick="setFilter('.cantbedone')">Can Do</button>
        <button class="notDoneButton button" onclick="setFilter('.isdone')">Not Done</button>
        <button class="AllButton button" onclick="setFilter('.tasks.none')">All</div>
    </div>

<script>
        window.addEventListener("load",renderAll());
        console.log("done")  
</script></body></html>
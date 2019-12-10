# dnd

```js
//iindex.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import '@atlaskit/css-reset';
import { DragDropContext, Droppable } from 'react-beautiful-dnd';
import styled from 'styled-components';
import initialData from './initial-data';
import Column from './column';
const Container = styled.div`
    display: flex; //align kolom next to each other
`;
class App extends Component {
    state = initialData;
    onDragStart = () => {
      document.body.style.color: 'orange', //ketika start dragging, color text jadi orange, tapi akan tetap orange setelah drag selesai, rubahnya lagi di onDragent func
      document.body.style.transition: 'background-color 0.2s ease',  //smoot 
    };
    onDragUpdate =(update) {
        const { destination } = update;
        const opacity = destination 
            ? destination.index/Object.keys(this.state.tasks).length
            : 0
            document.body.style.backgroundColor: `rgba(153,141,217, ${opacity})`;
    }
    onDragEnd = result => {
        //todo: reorder our column
        //data yg kita mau dari result obj hanya source, destinantion, droggabeId
        document.body.style.color: 'inherit'; //biar warna text balik hitam
        document.body.style.backgroundColor: 'inherit';
        const { source, destination, draggableId } = result;
        //cek konisi kalo ada terjadi perpindahan posisi, dgn 2 cek
        if (!destination) return;
        if (
            destination.droppableId === source.droppableId &&
            destination.index === source.index
        )
            return;
        //ambil data colum dari state
        const start = this.state.columns[source.droppableId]; //colunm-1 karena kolom cuma 1
        const finish = this.state.columns[destination.droppableId]; //colunm-1 karena kolom cuma 1
        if (start === finish) { //biar ketika di drop ke posisi baru dalam kolom yg sama, dia stay di tempat baru
            //bikin objeck baru, dari  paad mutating data  langgssungn
            const newTaskIds = Array.from(column.taskIds);
            //move new taskIds ke inddex barunya.
            //from surce.id  remove 1 item -> splicce
            newTaskIds.splice(source.index, 1);
            //from destinationn index remove nothing, add draggableId = taskid
            newTaskIds.splice(destination.index, 0, draggableId);
            //bikin kolom baru.. yg isinya yg olld dan newtask tadi
            const newColumn = {
                ...column,
                taskIds: newTaskIds
            };
            const newState = {
                ...this.state,
                columns: {
                    ...this.state.columns,
                    [newColumn.id]: newColumn
                }
            };
            this.setState(newState);
            return;
        }
        //approach jka start dan finish colun, berbeda
        //create new array dari pcolun dia start
        const finishTaskIds = Array.from(finish.taskIds); //contains taskids as the lash finished items
        finishTaskIds.splice(destination.index, 0, draggableId); //insert the draggableid at the destination index
        const newFinish = { //finish column with the new taskids for that column
            ...finish,
            taskIds: finishTaskIds
        };
        const startTaskIds = Array.from(start.taskIds); //contain same id from the old array
        startTaskIds.splice(source.index, 1); //remove the dragged taskid yg source dari array itu
        const newStart = {
            ...start, //same properties as the old colom
            taskIds: startTaskIds //tapi tanpa new start taskids array
        };
        const newState = {
            ...this.state, //propersi sama, tapi update columns nya jadi yg di bawah ini
            columns: {
                ...this.state.columns,
                [newStart.id]: newStart,
                [newFinish.id]: newFinish
            }
        };
        this.setState(newState);
    };
    render() {
        return (
            <DragDropContext 
                //onDragStart={this.onDragStart} 
                onDragEnd={this.onDragEnd}>
                    <Droppable droppableId="all-columns" direction="horizontal" type='colomn'> //droppableId bebas, horizontal mau reordering kolom2 ke kiri/kanan                       {() => (
                            {provided => (
                                <Container  //unt styling biar bisa berjejer kolomnya
                                innerRef={provided.innerRef} //tulis aja, dom ref
                                {...provided.droppableProps}> 
                                {this.state.columnOrder.map(columnId => {
                                    const column = this.state.columns[columnId];
                                    const tasks = column.taskIds.map(
                                        taskId => this.state.tasks[taskId]
                                    );
                                    return (
                                        <Column
                                            key={column.id}
                                            column={column}
                                            tasks={tasks}
                                        />
                                    );
                                })}
                                {provided.placeholder}
                            </Container>
                            )}
                        )}
                </Droppable>
            </DragDropContext>
        );
    }
}
// Put the things into the DOM!
ReactDOM.render(<App />, document.getElementById('root'));
//dragDropContext runtuk wrap semua
//droppabe; region untuk nge drag and drop
//draggable: component /list item yg  bisa dipindah2 dan di attach ke drappablle
//wrap semua dalam dragDropContext
// dia punya 3 cacllback.. onDragStart dianggil ketika mulai ngedrag
//onDragUpdate: ketika ada  yg pindah ke posisi baaru
///onDragEnd: dipanggil ketika nge drag selesai. * ini required disii syncronously update state kita.
/*
contoh data result
onDragEnd = result => {
        //todo: reorder our column
    };
const result = {
    draggableId : 'task-1',
        type: 'TYPE',
        reason: 'DROP',
        source: {
            droppableId: 'column-1',
            index: 0,
        },
        destinanttion: {
            droppableId: 'column-1',
            index: 1,
        }
}
*/
// untu more (global) style pake callback lain di DragDopContenx, onDrag Start, onDragUpdate.
//tapi biar gampang, styling rily on snapshot value aja. bukan di calbacks ini
//onDragStart
const start = {
    draggableId: 'task-1',
    type: 'TYPE',
    source: {
        droppableId: 'column-1',
        index: 0,
    }
}
//onDragUpdate
const update = {
    ...start,
    destination: {
        droppableId: 'column-1',
        index: 1,
    }
}
//onDragEnd
const result = {
    ...update,
    reason: 'DROP'
}
```


```js
//initial-data.js
// hanya perlu simple data struccture, unopinnionateed
// data basis untuk render our app
const initialData = {
    tasks: {
        'task-1': { id: 'task-1', content: 'take out the garbage' },
        'task-2': { id: 'task-2', content: 'watch a show' },
        'task-3': { id: 'task-3', content: 'charge phone' },
        'task-4': { id: 'task-4', content: 'cook dinner' }
    },
    columns: {
        'column-1': {
            id: 'column-1',
            title: 'To do',
            taskIds: ['task-1', 'task-2', 'task-3', 'task-4']
        },
        'column-2': {
            id: 'column-2',
            title: 'In Progress',
            taskIds: []
        },
        'column-3': {
            id: 'column-2',
            title: 'Done',
            taskIds: []
        }
    },
    columnOrder: ['column-1', 'column-2', 'column-3']
};
export default initialData;
//columnOrder unt record the order of the column
```

```js
//column.js
import React, { Component } from 'react';
import styled from 'styled-components';
import { Droppable } from 'react-beautiful-dnd';
import Task from './task';
const Container = styled.div`
    margin: 8px;
    border: 1px solid lightgrey;
    border-radius: 2px;
    width: 220px;
    /semuakolomjadisamawithnya
    display: flex; //biar kolom kedua dst bisa didrop smt
    flex-direction: column; //biar children are  align vertically
`;
const Title = styled.h3`
    padding: 8px;
`;
const TaskList = styled.div`
    padding: 8px;
    transition: background-color 0.2s ease; //for a smotth color changing
    background-color: ${props =>
        props.isDraggingOver
            ? 'skyblue'
            : 'white'}; //background to do list nya jadi blue kalo lagi ada yg di drag di kolom todo image-resolution, kalo nge dragnya sampe luber keluar area list to :double-button, warna jadi white.
    flex-grow: 1; //biar kolom grow to feel the available space
    min-height: 100px; //biar kalo kolom masih kosong, bakal possible to drop smt
`;
class innerList extends Component {
    shouldComponentUpdate(nextProps) {
        if (nextProps.tasks === this.props.tasks) {
            return false;
        }
        return true;
    }
    render() {
        return this.props.tasks.map((task, index) => (
            <Task key={task.id} task={task} index={index} />
        ));
    }
}
export default class Column extends Component {
    render() {
        return (
            <Container>
                <Title>{this.props.column.title}</Title>
                <Droppable droppableId={this.props.column.id}>
                    //props lain"direction:"horizontal""
                    {(provided, snapshot) => (
                        <TaskList
                            innerRef={provided.innerRef} //tulis aja, dom ref
                            {...provided.droppableProps}
                            isDraggingOver={snapshot.isDraggingOver}>
                            {/* {this.props.tasks.map((task, index) => (
                                <Task key={task.id} task={task} index={index} />
                            ))} */}{' '}
                            //ganti ke komponen baru unt should component update
                            <innerList task={this.props.tasks} />
                            {provided.placeholder}
                        </TaskList>
                    )}
                </Droppable>
            </Container>
        );
    }
}
//droppableId required dan musti unique, chilfd nya usti dipanggil dalam callback dengan 2 arguments:
//provided,: object yg punya props:  droppableProps
//insert a plaeholder penting, add sebagai cchild <TaskList>
// draggable snapshot data example
const draggableSnapshot = {
    isDragging: true, //when the draggable is currently being drag
    draggingOver: 'column-1' //set to id of the droppable, nilai bisa null
};
const droppableSnapshot = {
    isDraggingOver: true,
    draggingOverWith: 'task-1' // set to id of the dgraggable that is dragging over a droppable, null jika droppable is not being drag over
};


```

```js
// task.js
import React, { Component } from 'react';
import styled from 'styled-components';
import { Draggable } from 'react-beautiful-dnd';
const Container = styled.div`
    margin-bottom: 8px;
    border: 1px solid lightgrey;
    border-radius: 2px;
    padding: 8px;
    background-color: ${props =>
        props.isDragging
            ? 'lightgreen'
            : 'white'}; //ketika di drag, item jadi green
    //bikin container intu a flex parent
`;
const Handle = styled.div`
    width: 20px;
    height: 20px;
    background-color: orange;
    border-radius: 4px; //box kecil orange di kiri text items
    margin: 8px;
`;
export default class Task extends Component {
    render() {
        return (
            <Draggable
                draggableId={this.props.task.id}
                index={this.props.index}>
                {(provided, snapshot) => (
                    <Container
                        {...provided.draggableProps}
                        // {...provided.dragHandleProps} //handle all entire draggable nge drap di titik manapun dari draggable/particular item, bisa
                        innerRef={provided.innerRef}
                        isDragging={snapshot.isDragging}>
                        <Handle {...provided.dragHandleProps} /> //pindah props
                        kesini biar ketika di titik yg ada kotak orangenya aja
                        yg bisa bikin dragging beerfungsi, di tulisan ga fungsi
                        lagi.
                        {this.props.task.content}
                    </Container>
                )}
            </Draggable>
        );
    }
}
//child nya draggable musti a functionn
// add ini {...provided.draggableProps}
// {...provided.dragHandleProps}
// innerRef={provided.innerRef}> biar bisa  di dragg itemnya atas ke bawah mppake mouse, keyboard
//ampe sini reordering ganti2 posisi itemnya ga presere.. untu itu masukkan logic ke fucntion onDraggEnd
//snapshot itu untuk style component during drag and drop
```

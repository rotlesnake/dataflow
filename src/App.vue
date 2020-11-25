<template>
<div id="app">

    <dataflow @contextmenu.native="showContextMenu" @click.native="closeContextMenu" ref="container" :blocksContent="blocks" :scene.sync="scene" @blockSelect="selectBlock" @blockDeselect="deselectBlock" class="dataflow-container" />

	<ul id="contextMenu" ref="contextMenu" tabindex="-1" v-show="contextMenu.isShow" @blur="closeContextMenu" :style="{top: contextMenu.top + 'px', left: contextMenu.left + 'px'}">
		<template v-for="item in blocks">
			<div :key="item.id">
				<li class="label"> {{item.family}} </li>
				<li class="name" @click="addBlockContextMenu(item.name)"> {{item.title || item.name}}</li>
			</div>
		</template>
	</ul>

</div>
</template>

<script>
import dataflow from './components/dataflow.js'
import './components/dataflow.css'

export default {
    components: {
        dataflow
    },
    data() {
        return {
            blocks: [
				{
					id: 1,
                    family: 'Формы',
                    name: 'stage_solution',
                    title: 'Согласование',
                    width: 300,
                    fields: [],
                    inputs: [
						{name: '',label: 'Открыть форму',type: 'event',color: '#00ff00',shape: 'circle'},
                        {name: '',label: 'Поля ввода данных',type: 'field',color: '#0000ff',shape: 'square'},
                    ],
                    outputs: [
						{name: '',label: 'Согласовано',type: 'event',color: '#00ff00'},
                        {name: '',label: 'Отклонено',type: 'event',color: '#ff0000'},
                    ]
                },
                {
					id: 2,
                    family: "Поля ввода",
                    name: "field_enter_string",
                    title: "Ввод строки",
                    width: 200,
                    fields: [],
                    inputs: [],
                    outputs: [
						{label: "Ввод поля",type: 'field',color: '#0000ff',shape: 'square'},
					],
                },
            ],
            scene: {
                blocks: [],
                links: [],
                container: {
                    centerX: 1042,
                    centerY: 140,
                    scale: 1
                }
            },
            selectedBlock: null,
            selectedType: 'delay',
            contextMenu: {
                isShow: false,
                mouseX: 0,
                mouseY: 0,
                top: 0,
                left: 0
            }
        }
    },

    methods: {
        selectBlock(block) {
            this.selectedBlock = block
        },
        deselectBlock() {
            this.selectedBlock = null
        },
        filteredBlocks(type) {
            return this.blocks.filter(value => {
                return value.family === type
            })
        },
        addBlock() {
            this.$refs.container.addNewBlock(this.selectedType)
        },

        showContextMenu(e) {
            if (e.preventDefault) e.preventDefault()

            this.contextMenu.isShow = true
            this.contextMenu.mouseX = e.x
            this.contextMenu.mouseY = e.y

            this.$nextTick(function () {
                this.setMenu(this.contextMenu.mouseY, this.contextMenu.mouseX)
                this.$refs.contextMenu.focus()
            })
        },

        getMousePosition(element, event) {
            let mouseX = event.pageX || event.clientX + document.documentElement.scrollLeft;
            let mouseY = event.pageY || event.clientY + document.documentElement.scrollTop;

            let offset = this.getOffsetRect(element);
            let x = mouseX - offset.left;
            let y = mouseY - offset.top;

            return {
                x: x,
                y: y
            };
        },
        getOffsetRect(element) {
            let box = element.getBoundingClientRect()

            let scrollTop = window.pageYOffset
            let scrollLeft = window.pageXOffset

            let top = box.top + scrollTop
            let left = box.left + scrollLeft

            return {
                top: Math.round(top),
                left: Math.round(left)
            }
        },

        setMenu(top, left) {
            let border = 5
            let contextMenuEl = this.$refs.contextMenu
            let containerElRect = this.$refs.container.$el.getBoundingClientRect()
            let largestWidth = containerElRect.right - contextMenuEl.offsetWidth - border
            let largestHeight = containerElRect.bottom - contextMenuEl.offsetHeight - border

            top -= containerElRect.y;
            left -= containerElRect.x;

            if (left > largestWidth) left = largestWidth
            if (top > largestHeight) top = largestHeight

            this.contextMenu.top = top
            this.contextMenu.left = left
        },
        addBlockContextMenu(name) {
            let offset = this.getOffsetRect(this.$refs.container.$el)
            let x = this.contextMenu.mouseX - offset.left
            let y = this.contextMenu.mouseY - offset.top

            this.$refs.container.addNewBlock(name, x, y)
            this.closeContextMenu()
        },
        closeContextMenu() {
            this.contextMenu.isShow = false
        }
    },
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;

    width: 98vw;
    height: 98vh;
    margin: 5px;
}

.dataflow-container {
    background: #eee;
    width: 100%;
    height: 100%;
    border: 1px solid black;
}

#contextMenu {
	min-width: 250px;
    position: absolute;
    z-index: 1000;
    background: white;
    border: 1px solid black;
    padding: 5px;
    margin: 0;
}

#contextMenu li.label {
    background: #f0f0f0;
    color: #999;
    font-size: 90%;
    list-style: none;
    text-align: center;
    margin-top: 5px;
}

#contextMenu li.name {
    cursor: pointer;
    list-style: none;
    color: #000;
}

#contextMenu li.name:hover {
    color: #000055;
}
</style>

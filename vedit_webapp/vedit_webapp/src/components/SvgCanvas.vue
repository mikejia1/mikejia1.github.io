<template>
    <div class="hello">
        <h1>{{ msg }}</h1>
        <!-- https://www.petercollingridge.co.uk/tutorials/svg/interactive/javascript/ -->
        <div style="border-style:solid">
            <div> Upload an SVG file. </div>
            <input type="file" ref="doc" @change="readFile()" />
            <div><br /></div>
        </div>
        <div style="border-style:solid">
            <div> Import svg from a link. </div>
            <input type="text" v-model="url" />
            <input type="button" value="submit url link" v-on:click="readUrl()" />
            <div>{{content}}<br /></div>
        </div>
        <div style="border-style:solid">
            <div>
                You can import multiple svgs at a time by supplying an API. Here is a sample:
                <p style="font-family:courier;">https://mocki.io/v1/65edb577-03e3-49b6-a449-2da55050bd0f</p>
                Create your own API at <a href="https://mocki.io">mock.io</a>.
            </div>
            <input type="text" v-model="api" />
            <input type="button" value="submit API link" v-on:click="fetchAPI()" />
            <div><br /></div>
        </div>
        <input type="button" value="rotate svg ordering" v-on:click="rotateOrder()" />
    </div>
    <!--does this canvas need to be an svg? -->
    <div style="border-style:solid">
        <svg xmlns="http://www.w3.org/2000/svg"
             id="canvas"
             viewBox="0 0 600 400"
             @load="makeDraggable">
            <svg xmlns="http://www.w3.org/2000/svg" v-for="(svg, index) in svgFiles" v-bind:key="index" v-html="svg" />
            <rect class="draggable"
                  fill="#b17bff" x="4" y="1" width="30" height="50" transform="translate(10, 0)" />
        </svg>
    </div>
</template>

<script>
    export default {
        name: 'SvgCanvas',
        props: {
            msg: String
        },
        data: () => ({ file: null, content: null, url: null, api: null, svgFiles: [], }),
        methods: {
            //https://masteringjs.io/tutorials/vue/file
            readFile() {
                this.file = this.$refs.doc.files[0];
                const reader = new FileReader();
                if (this.file.name.includes(".svg")) {
                    this.content = "check the console for file output";
                    reader.onload = (res) => {
                        this.svgFiles.push(res.target.result);
                    };
                    reader.onerror = (err) => console.log(err);
                    reader.readAsText(this.file);
                } else {
                    this.content = "please select an svg file";
                }
            },
            readUrl() {
                fetch("https://proxy.cors.sh/" + this.url).then(res => res.blob())
                    .then(blob => {
                        blob.text().then(res => {
                            this.svgFiles.push(res);
                        })
                    })
            },

            fetchAPI() {
                fetch(this.api)
                    .then((response) => response.json())
                    .then((data) => {
                        data.assets.forEach((item) => {
                            this.url = item.url;
                            this.readUrl();
                            this.url = null;
                        })
                    });

            },

            rotateOrder() {
                var temp = this.svgFiles.pop();
                this.svgFiles.unshift(temp);
            },

            makeDraggable(evt) {
                var svg = evt.target;
                console.log(svg + "makeDraggable");
                svg.addEventListener('mousedown', startDrag);
                svg.addEventListener('mousemove', drag);
                svg.addEventListener('mouseup', endDrag);
                svg.addEventListener('mouseleave', endDrag);
                function getMousePosition(evt) {
                    var CTM = svg.getScreenCTM();
                    if (evt.touches) { evt = evt.touches[0]; }
                    return {
                        x: (evt.clientX - CTM.e) / CTM.a,
                        y: (evt.clientY - CTM.f) / CTM.d
                    };
                }
                var selectedElement, offset, transform;
                function startDrag(evt) {
                    selectedElement = evt.target;
                    if (selectedElement.id == "canvas") return;
                    offset = getMousePosition(evt);
                    // Make sure the first transform on the element is a translate transform
                    var transforms = selectedElement.transform.baseVal;
                    if (transforms.length === 0 || transforms.getItem(0).type !== SVGTransform.SVG_TRANSFORM_TRANSLATE) {
                        // Create an transform that translates by (0, 0)
                        var translate = svg.createSVGTransform();
                        translate.setTranslate(0, 0);
                        // This will insert the translate as the first element of the transform. Transforms are just applied from right to left.
                        selectedElement.transform.baseVal.insertItemBefore(translate, 0);
                    }
                    // Get initial translation
                    transform = transforms.getItem(0);
                    offset.x -= transform.matrix.e;
                    offset.y -= transform.matrix.f;

                }

                function drag(evt) {
                    if (selectedElement) {
                        evt.preventDefault();
                        var coord = getMousePosition(evt);
                        transform.setTranslate(coord.x - offset.x, coord.y - offset.y);
                    }
                }

                function endDrag() {
                    selectedElement = false;
                    transform = false;
                    offset = false;
                }

            },


        },
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    .draggable {
        cursor: move;
    }
</style>

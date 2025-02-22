<template>
    <div id="desc-container">
        <h2 style="text-align: center;">Smart Image Editor</h2>
        <div class="description-container e-card">
            <div class='e-card-content '>
                <p>
                    The Image Editor, enhanced with AI, includes features like <i>Magic Eraser</i>, <i>Background
                        Changer</i> and <i>Background Remover</i>. Click the Sidebar to explore these capabilities. The
                    Image Editor uses the Stable Diffusion API for its AI features. Need to configure the Key in Sample.
                    Know more <a
                        href="https://github.com/syncfusion/smart-ai-samples/blob/master/vue/src/ai-components/ai-image-editor/Readme.md">here</a>.
                </p>
            </div>
        </div>
    </div>
    <div class="container">
        <div id="wrapper-container" ref="wrapperDiv">
            <div class="magic-eraser">
                <div class="upper-div">
                    <label>Magic Eraser</label>
                    <ejs-button id="remove-btn" aria-label="button" @click="removeBtnClick" iconCss='e-icons e-close'
                        cssClass='e-small e-round' :isPrimary="true"></ejs-button>
                </div>
                <div class="lower-div">
                    <video :muted="true" :loop="true" :autoPlay="true" preload="auto"
                        src="https://www.fotor.com/photo-editor-app/ai/video/removerobj.mp4"
                        style=' width: "130px"; height: 130px '></video>
                    <br></br><span>Select the object to be removed</span>
                </div>
                <ejs-button class="remove-button" id="eraseBtn" :isPrimary="true" @click="eraseBtnClick">Remove
                </ejs-button>
            </div>
            <div class="bg-changer">
                <div class="upper-div">
                    <label>Change Background</label>
                    <ejs-button id="bg-change-remove-btn" aria-label="button" @click="bgBtnClick"
                        iconCss='e-icons e-close' cssClass='e-small e-round' :isPrimary="true"></ejs-button>
                </div>
                <div class="lower-div">
                    <div class="col-lg-12 control-section">
                        <div id="colorpicker-control">
                            <span style=' margin-bottom: "10px" '>New Color</span>
                            <br></br>
                            <ejs-colorpicker id="color-picker" type="color" change={change} />
                        </div>
                    </div>
                    <br></br><span>Preset Colors</span>
                    <div class="col-xs-12 col-sm-12 col-lg-6 col-md-6">
                        <ejs-colorpicker ref="colorpickerObj" id="circle-palette" type="color" mode="Palette"
                            :modeSwitcher="false" :inline="true" :showButtons="false" :columns="6" :presetColors="{
                                'custom'
                                    : ['#f44336', '#e91e63', '#9c27b0', '#673ab7', '#2196f3', '#03a9f4', '#00bcd4'
                                        , '#009688', '#8bc34a', '#cddc39', '#ffeb3b', '#ffc107']
                            }" :beforeTileRender="beforeTileRender" :change="change" />
                    </div>
                    <span>Custom Background</span><br></br>
                    <ejs-textbox id="outlined" ref="textbox" placeholder='Example: Waterfalls, Mountains, etc..'
                        cssClass='e-outline' />
                </div>
                <ejs-button class="bg-change-button" id="bgChangeBtn" @click="bgChangeBtnClick"
                    :isPrimary="true">Apply</ejs-button>
            </div>
            <div id="sidebar-wrapper" class="control-section">
                <div style='width: "100%" '>
                    <ejs-toolbar id="defaultToolbar" cssClass="defaultToolbar" height="50px" :clicked="toolbarCliked">
                        <e-items>
                            <e-item prefixIcon="e-tbar-menu-icon tb-icons" tooltipText="Menu" />
                            <e-item template="folderEle" cssClass="e-folder" />
                        </e-items>
                        <template #folderEle>
                            <div class="e-folder">
                                <div class="e-folder-name">AI Image Editor</div>
                            </div>
                        </template>
                    </ejs-toolbar>
                </div>
                <div class="maincontent" style='width: "100%" '>
                    <div id="controlWrapper">
                        <ejs-imageeditor id="imageeditor" class="row" ref="imageEditor"
                            :fileOpened="timeOut"></ejs-imageeditor>
                    </div>
                </div>
            </div>
            <ejs-sidebar id="defaultSidebar" class="default-sidebar" ref="sideObj" width="200px" target=".maincontent"
                position='Left' type='Push'>
                <ejs-treeview id="defaultTree" cssClass="image-editor-tree" ref="treeObj" :nodeSelected='OnSelect'
                    :fields='{
                        dataSource: treeData, id: "id", text: "name", selected: "selected", parentID: "pid",
                        hasChildren: "hasChild", expanded: "expanded"
                    }'></ejs-treeview>
            </ejs-sidebar>
        </div>
    </div>
</template>
<script>
import { ImageEditorComponent } from "@syncfusion/ej2-vue-image-editor"
import { ColorPickerComponent, TextBoxComponent } from "@syncfusion/ej2-vue-inputs";
import { ItemDirective, ItemsDirective, SidebarComponent, ToolbarComponent, TreeViewComponent } from "@syncfusion/ej2-vue-navigations";
import { hideSpinner, showSpinner } from "@syncfusion/ej2-vue-popups";
import { createElement, Draggable } from "@syncfusion/ej2-base";
import { ButtonComponent } from "@syncfusion/ej2-vue-buttons";
import { StabilityAiModel, StabilityAiModelBGRemover, StabilityAiModelMagicEraser } from '../common/stability-model';

export default {
    components: {
        "ejs-imageeditor": ImageEditorComponent,
        "ejs-colorpicker": ColorPickerComponent,
        "ejs-textbox": TextBoxComponent,
        "ejs-sidebar": SidebarComponent,
        "ejs-toolbar": ToolbarComponent,
        "ejs-treeview": TreeViewComponent,
        "ejs-button": ButtonComponent,
        "e-item": ItemDirective,
        "e-items": ItemsDirective
    },
    data() {
        return {
            treeData: [
                { id: "1", name: "Magic Eraser", imageUrl: "https://www.fotor.com/photo-editor-app/ai/thumb/remove%20object.jpg" },
                { id: "2", name: "Change Background", imageUrl: "https://www.fotor.com/photo-editor-app/ai/thumb/changeBg1_1.jpg" },
                { id: "2", name: "Remove Background", imageUrl: "https://www.fotor.com/photo-editor-app/ai/thumb/bgremover.jpg" }
            ],
        }
    },
    methods: {
        beforeTileRender: function (args) {
            args.element.classList.add('e-circle-palette');
            args.element.appendChild(createElement('span', { class: 'e-circle-selection' }));
        },
        timeOut: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            setTimeout(() => {
                imageEditorObj.update();
            }, 200);
        },

        imageDataToBase64: function (imageData) {
            const canvas = document.createElement('canvas');
            canvas.width = imageData.width;
            canvas.height = imageData.height;

            const ctx = canvas.getContext('2d');
            if (ctx) {
                ctx.putImageData(imageData, 0, 0);
                return canvas.toDataURL();
            }
            return '';
        },

        toggleDisplay: function (elementClassToShow, elementClassToHide) {
            (document.getElementsByClassName(elementClassToHide)[0]).style.display = 'none';
            (document.getElementsByClassName(elementClassToShow)[0]).style.display = 'block';
        },

        processImageData: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const wrapperDiv = this.$refs.wrapperDiv;
            showSpinner(imageEditorObj.element);
            wrapperDiv.style.opacity = '0.5';
            let imageData = (imageEditorObj).getImageData(false);
            let url = this.imageDataToBase64(imageData);
            const file = this.base64ToFile(url, 'image.png');
            this.removeBG(file);
        },

        OnSelect: function (args) {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const treeObj = this.$refs.treeObj.ej2Instances;
            switch (args.nodeData.text) {
                case "Magic Eraser":
                    this.toggleDisplay('magic-eraser', 'bg-changer');
                    imageEditorObj.update();
                    imageEditorObj.element.setAttribute('data-value', 'mask-drawing');
                    (imageEditorObj).freehandDraw(true);
                    treeObj.selectedNodes = [];
                    break;
                case "Change Background":
                    this.toggleDisplay('bg-changer', 'magic-eraser');
                    treeObj.selectedNodes = [];
                    this.processImageData();
                    break;
                case "Remove Background":
                    this.processImageData();
                    break;
            }
        },

        bgRemoveBtnClick: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const wrapperDiv = this.$refs.wrapperDiv;
            (document.getElementsByClassName('bg-changer')[0]).style.display = 'none';
            colorPicker.refresh();
            colorPickerVal = '#ffffff';
            outlineTextBox.value = '';
            const selectedElement = (colorPicker.element.parentElement).querySelector('.e-selected');
            if (selectedElement) {
                selectedElement.classList.remove('e-selected');
            }
            hideSpinner(imageEditorObj.element);
            wrapperDiv.style.opacity = '1';
        },

        getImageDataAsBase64: function (imageEditor) {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const imageData = imageEditorObj.getImageData(false);
            return this.imageDataToBase64(imageData);
        },

        base64ToFile: function (base64String, fileName) {
            const byteString = atob(base64String.split(',')[1]);
            const arrayBuffer = new ArrayBuffer(byteString.length);
            const intArray = new Uint8Array(arrayBuffer);
            for (let i = 0; i < byteString.length; i++) {
                intArray[i] = byteString.charCodeAt(i);
            }
            const blob = new Blob([intArray], { type: 'image/png' });
            const file = new File([blob], fileName, { type: 'image/png' });
            return file;
        },

        removeBG: function (file) {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const treeObj = this.$refs.treeObj.ej2Instances;
            const wrapperDiv = this.$refs.wrapperDiv;
            let aiOutput = StabilityAiModelBGRemover(file);
            aiOutput.then((result) => {
                imageEditorObj.open(result, false);
                setTimeout(() => {
                    hideSpinner(imageEditorObj.element);
                    wrapperDiv.style.opacity = '1';
                    treeObj.selectedNodes = [];
                }, 100);
            });
        },

        removeBtnClick: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const wrapperDiv = this.$refs.wrapperDiv;
            imageEditorObj.element.setAttribute('data-value', '');
            (document.getElementsByClassName('magic-eraser')[0]).style.display = 'none';
            hideSpinner(imageEditorObj.element);
            wrapperDiv.style.opacity = '1';
            imageEditorObj.discard();
        },

        bgBtnClick: function () {
            bgRemoveBtnClick();
        },

        eraseBtnClick: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const treeObj = this.$refs.treeObj.ej2Instances;
            const maskUrl = this.getImageDataAsBase64(imageEditorObj);
            imageEditorObj.element.setAttribute('data-value', '');
            imageEditorObj.freehandDraw(false);
            const url = this.getImageDataAsBase64(imageEditorObj);
            showSpinner(imageEditorObj.element);
            const file = this.base64ToFile(url, 'image.png');
            const maskFile = this.base64ToFile(maskUrl, 'mask.png');
            const aiOutput = StabilityAiModelMagicEraser(file, maskFile);
            aiOutput.then((result) => {
                imageEditorObj.open(result, false);
                setTimeout(() => {
                    hideSpinner(imageEditorObj.element);
                    wrapperDiv.style.opacity = '1';
                    treeObj.selectedNodes = [];
                }, 100);
                (document.getElementsByClassName('magic-eraser')[0]).style.display = 'none';
            });
        },

        bgChangeBtnClick: function () {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            const wrapperDiv = this.$refs.wrapperDiv;
            showSpinner(imageEditorObj.element);
            wrapperDiv.style.opacity = '0.5';
            if (outlineTextBox.value && outlineTextBox.value !== '') {
                let url = getImageDataAsBase64(imageEditorObj);
                const file = base64ToFile(url, 'image.png');
                let prompt = outlineTextBox.value;
                let searchPrompt = 'Background of the image';
                let aiOutput = StabilityAiModel(file, prompt, searchPrompt);
                aiOutput.then((result) => {
                    imageEditorObj.open(result, false);
                    setTimeout(() => {
                        bgRemoveBtnClick();
                    }, 100);
                    (document.getElementsByClassName('bg-changer')[0]).style.display = 'none';
                });
            } else {
                bgRemoveBtnClick();
            }
        },

        change: function (args) {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            colorPickerVal = args.currentValue.hex;
            imageEditorObj.open('', false, { backgroundColor: colorPickerVal });
        },

        toolbarCliked: function (args) {
            const imageEditorObj = this.$refs.imageEditor.ej2Instances;
            if (args.item.tooltipText == "Menu") {
                this.$refs.sideObj.toggle();
                setTimeout(() => {
                    imageEditorObj.update();
                }, 500);
            }
        }


    },
    mounted() {
        const imageEditorObj = this.$refs.imageEditor.ej2Instances;
        const draggableElements = ['magic-eraser', 'bg-changer'];
        draggableElements.forEach(className => {
            const dragElement = document.getElementsByClassName(className)[0];
            new Draggable(dragElement, { clone: false });
        });
        imageEditorObj.open('https://images.pexels.com/photos/103123/pexels-photo-103123.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1');
    }
}
</script>

<style scoped>
#remove-btn,
#bg-change-remove-btn {
    background: #fff;
    color: #000;
}

.e-list-item .e-fullrow {
    width: 100%;
    height: 80%;
}

.magic-eraser,
.bg-changer {
    width: 280px;
    border-radius: 10px;
    position: absolute;
    left: 10px;
    top: 55px;
    z-index: 1000;
    background: #fff;
    display: none;
    border: 1px solid #ccc;
}

.bg-changer {
    top: 55px;
}

.upper-div {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    border-bottom: 1px solid #ccc;
    font-size: 16px;
    font-family: "Roboto", -apple-system, BlinkMacSystemFont, "Segoe UI", "Helvetica Neue", sans-serif;
}

hr {
    border: 0;
    height: 1px;
    background: #ccc;
}

.lower-div {
    padding: 20px;
    background-color: #fff;
    border-bottom: 1px solid #ccc;
    font-size: 14px;
    font-family: "Roboto", -apple-system, BlinkMacSystemFont, "Segoe UI", "Helvetica Neue", sans-serif;
}

.remove-button,
.bg-change-button {
    display: block;
    margin: 10px auto;
    padding: 10px 20px;
    color: white;
    border: none;
    cursor: pointer;
    width: 200px !important;
}

#controlWrapper {
    width: 50vw;
    height: 600px;
}

#sidebar-wrapper.control-section {
    padding: 0;
}

#sidebar-wrapper .default-sidebar {
    z-index: 20 !important;
}

#sidebar-wrapper .e-content-animation {
    width: auto;
}

@font-face {
    font-family: 'Material_toolbar';
    src: url(data:application/x-font-ttf;charset=utf-8;base64,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) format('truetype');
    font-weight: normal;
    font-style: normal;
}

.e-tbar-btn .tb-icons {
    font-family: 'Material_toolbar';
    speak: none;
    font-size: 16px;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
}

.e-toolbar .e-icons {
    font-size: 20px;
}

.e-tbar-menu-icon:before {
    content: "\e718";
}

#defaultTree.e-treeview .e-list-icon,
.e-treeview .e-list-img {
    width: 150px !important;
    height: 130px !important;
}

#defaultTree .e-treeview .e-fullrow {
    height: 162px !important;
}

#defaultTree .e-text-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 0 !important;
}


/* Color Picker Code Starts */
.e-container .e-palette .e-circle-palette {
    border: 0;
    height: 32px;
    width: 32px;
    border-radius: 20px;
    margin: 4px;
}

.e-bigger .e-container .e-palette .e-circle-palette {
    height: 32px;
    width: 32px;
}

.e-container .e-palette .e-circle-palette:hover {
    box-shadow: none;
    transform: scale(1.2);
    transition: transform .2s ease-out;
}

.e-circle-palette .e-circle-selection {
    height: 32px;
    width: 32px;
    border-radius: 20px;
    display: inline-block;
    transform: scale(0);
    transition: transform 1.2s ease-in;
}

.e-circle-palette.e-selected .e-circle-selection {
    transform: scale(0.8);
    background-color: #fff;
    transition: transform .2s ease-out;
}

#circle-palette+.e-container {
    background-color: transparent;
    border-color: transparent;
    box-shadow: none;
}

.e-container .e-palette .e-circle-palette.e-selected {
    outline: none;
}

.e-circle-wrap {
    width: 22%;
}

/* Color Picker Code Ends*/

.e-input-group {
    margin-top: 10px !important;
}

.bg-changer .lower-div span {
    font-weight: 500;
}

#defaultTree .e-list-item {
    padding: 10px;
    border: 2px solid transparent;
}

#defaultTree .e-list-item:hover {
    border: 2px solid lightgray;
}

#defaultTree .e-fullrow {
    left: 20px !important;
    width: 160px !important;
}

#defaultTree.e-treeview .e-list-icon,
#defaultTree.e-treeview .e-list-img {
    margin: 0 !important;
}

#defaultTree.e-treeview .e-ul {
    margin-right: 10px;
    margin-top: 10px;
}

#colorpicker-control .e-colorpicker-wrapper {
    margin-top: 10px;
}

.e-palette {
    padding-left: 0px !important;
}

.defaultToolbar.e-toolbar .e-folder {
    font-weight: 500;
    font-size: 16px;
}

.magic-eraser .lower-div {
    text-align: center;
}
</style>
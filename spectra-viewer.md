---
layout: page
full-width: true
# title: Spectra Viewer
show-avatar: false
permalink: /spectra-viewer/
---

<script src="https://cdn.jsdelivr.net/npm/@panzoom/panzoom/dist/panzoom.min.js"></script>

<h2 style="margin-top: -30px; ">Spectra Viewer</h2>

<p style="margin-top: 0; margin-bottom: 2px;">
Run:
<select id="runSelect"></select>
<button onclick="changeRun(-1)">◀</button>
<button onclick="changeRun(1)">▶</button>

&nbsp; / &nbsp;

Slit:
<select id="slitSelect"></select>
<button onclick="changeSlit(-1)">◀</button>
<button onclick="changeSlit(1)">▶</button>
</p>

<div style="display:flex;gap:15px;align-items:flex-start;">

<div style="flex:1;min-width:0;">
<!-- <h3>Best Fit Spectrum</h3> -->
<div id="specContainer"
     style="min-height:100px;border:1px solid #000000;overflow:hidden;cursor:grab;">
<img id="specImg" draggable="false">
</div>
</div>

<div style="flex:1;min-width:0;">
<!-- <h3>Corner Plot</h3> -->
<div id="cornerContainer"
     style="min-height:100px;border:1px solid #000000;overflow:hidden;cursor:grab;">
<img id="cornerImg" draggable="false">
</div>
</div>

</div>

<script>

let data;

fetch("/assets/data/spectra.json")
.then(r=>r.json())
.then(json=>{

    data=json;

    let run=document.getElementById("runSelect");

    Object.keys(data).forEach(r=>{

        let o=document.createElement("option");
        o.value=r;
        o.text=r;
        run.appendChild(o);

    });

    run.onchange=updateSlits;

    updateSlits();

});

function updateSlits(){

    let run=document.getElementById("runSelect").value;

    let slit=document.getElementById("slitSelect");

    slit.innerHTML="";

    Object.keys(data[run]).forEach(s=>{

        let o=document.createElement("option");
        o.value=s;
        o.text=s;
        slit.appendChild(o);

    });

    slit.onchange=showImages;

    showImages();

}

function changeRun(direction){

    let run=document.getElementById("runSelect");

    let newIndex=run.selectedIndex + direction;

    if(newIndex < 0){
        newIndex = run.options.length - 1;
    }

    if(newIndex >= run.options.length){
        newIndex = 0;
    }

    run.selectedIndex=newIndex;

    updateSlits();

}


function changeSlit(direction){

    let slit=document.getElementById("slitSelect");

    let newIndex=slit.selectedIndex + direction;

    if(newIndex < 0){
        newIndex = slit.options.length - 1;
    }

    if(newIndex >= slit.options.length){
        newIndex = 0;
    }

    slit.selectedIndex=newIndex;

    showImages();

}

function resetPanzoom(container,img,url,fitWidth){

    img.src=url;

    img.onload=()=>{

        container.innerHTML="";
        img.style.display="block";
        img.style.transformOrigin="0 0";

        if(fitWidth){
            // 左图：适应宽度，可变高度
            img.style.maxWidth="";
            img.style.width="100%";
            img.style.height="auto";
        } else {
            // 右图：保持原始像素大小，只显示左上角一部分
            img.style.maxWidth="none";
            img.style.width="200%";
            img.style.height="auto";
        }

        container.appendChild(img);

        const panzoom = Panzoom(img,{
            maxScale:100,
            minScale:0.01
            // 不设startScale，默认就是1，对应上面两种情况各自的"原始状态"
        });

        container.addEventListener('wheel', panzoom.zoomWithWheel);

        img.ondblclick=()=>{
            panzoom.reset(); // 回到各自的初始状态（适应宽度 或 原始像素大小）
        };

    };

}

function showImages(){

    let run=document.getElementById("runSelect").value;
    let slit=document.getElementById("slitSelect").value;

    resetPanzoom(
        document.getElementById("specContainer"),
        document.getElementById("specImg"),
        data[run][slit].spec,
        true   // 左图：适应宽度
    );

    resetPanzoom(
        document.getElementById("cornerContainer"),
        document.getElementById("cornerImg"),
        data[run][slit].corner,
        false  // 右图：原始像素大小
    );

}

</script>
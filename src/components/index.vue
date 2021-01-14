<template>
  <div class="index">
    <div class="function-list">
      <div
        v-for="(item, index) in funcArry"
        :key="index"
        @click="choseFun(index)"
        class="fun-name"
      >
        {{ item }}
      </div>
    </div>
    <div id="father" class="canvas-box">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        id="svg"
        version="1.1"
        height="600"
        width="1000"
      >
        <defs>
          <marker
            id="idArrow"
            viewBox="0 0 20 20"
            refX="20"
            refY="10"
            markerUnits="strokeWidth"
            markerWidth="5"
            markerHeight="13"
            orient="auto"
          >
            <path d="M 0 0 L 20 10 L 0 20 z" fill="#eee" stroke="#eee" />
          </marker>
        </defs>
      </svg>
      <div
        v-for="(item, index) in choseArry"
        :key="index"
        class="block"
        @dblclick="deleteBox(index, $event)"
      >
        <div class="point-box">
          <span class="point point-top"> </span>
          <span class="point point-bottom"> </span>
        </div>
        {{ item }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
      fLeft: 0,
      fTop: 0,
      odiv: "",
      currentLine: "",
      startPointDom: "",
      endPointDom: "",
      funcArry: ["功能1", "功能2", "功能3", "功能4", "功能5"],
      choseArry: []
    };
  },
  mounted() {
    this.init();
    let fdiv = document.getElementById("father");
    this.fTop = fdiv.offsetTop;
    this.fLeft = fdiv.offsetLeft;
  },
  methods: {
    deleteBox(i, event) {
      let deleteDom = event.target.children[0].children;
      for (let i = 0; i < deleteDom.length; i++) {
        let startId = deleteDom[i].getAttribute("data-start")
          ? deleteDom[i].getAttribute("data-start").split(",")
          : [];
        let endId = deleteDom[i].getAttribute("data-end")
          ? deleteDom[i].getAttribute("data-end").split(",")
          : [];
        if (startId[0]) {
          for (let j = 0; j < startId.length; j++) {
            let lineDom = document.getElementById(startId[j]);
            document.getElementById("svg").removeChild(lineDom);
          }
        }
        if (endId[0]) {
          for (let j = 0; j < endId.length; j++) {
            let lineDom = document.getElementById(endId[j]);
            document.getElementById("svg").removeChild(lineDom);
          }
        }
      }
      setTimeout(() => {
        this.choseArry.splice(i, 1);
      }, 500);
    },
    choseFun(i) {
      this.choseArry.push(this.funcArry[i]);
    },
    init() {
      let _this = this;
      document.getElementById("father").onmousedown = function(evn) {
        _this.odiv = evn.target;
        if (evn.target.classList[0] === "block") {
          _this.dragBlock();
        } else if (evn.target.classList[0] === "point") {
          _this.startPointDom = evn.target;
          let changeEle = document.createElementNS(
            "http://www.w3.org/2000/svg",
            "path"
          );
          _this.dragLine(changeEle, evn.clientX, evn.clientY);
        }
      };
      document.onmouseup = function(evn) {
        if (_this.currentLine !== "") {
          if (evn.target.classList[0] !== "point") {
            document.getElementById("svg").removeChild(_this.currentLine);
          } else {
            _this.endPointDom = evn.target;
            let startId = _this.startPointDom.getAttribute("data-start")
              ? _this.startPointDom.getAttribute("data-start").split(",")
              : [];
            let endId = _this.endPointDom.getAttribute("data-end")
              ? _this.endPointDom.getAttribute("data-end").split(",")
              : [];
            startId.push(_this.currentLine.getAttribute("id"));
            endId.push(_this.currentLine.getAttribute("id"));
            _this.startPointDom.setAttribute("data-start", startId.join(","));
            _this.endPointDom.setAttribute("data-end", endId.join(","));
          }
        }
        _this.currentLine = "";
        _this.odiv = "";
        document.onmousemove = null;
      };
    },
    createCPath(x1, y1, x2, y2) {
      var path = "M" + x1 + " " + y1 + " ";
      var cx1 = x1;
      var cy1 = (y1 + y2) / 2;
      var cx2 = x2;
      var cy2 = (y1 + y2) / 5;
      var c = "C" + cx1 + " " + cy1 + "," + cx2 + " " + cy2 + ",";
      path = path + c + x2 + " " + y2;
      return path;
    },
    dragLine(pathDom, starOne, starTwo) {
      let _this = this;
      this.currentLine = pathDom;
      document.onmousemove = function(ev) {
        //IE支持event，firefox不支持
        var oEvent = ev || event;
        var scrollLeft =
          document.documentElement.scrollLeft || document.body.scrollLeft;
        var scrollTop =
          document.documentElement.scrollTop || document.body.scrollTop;
        var starttop = starTwo + scrollTop - _this.fTop;
        var startleft = starOne + scrollLeft - _this.fLeft;
        var endtop = oEvent.clientY + scrollTop - _this.fTop;
        var endleft = oEvent.clientX + scrollLeft - _this.fLeft;
        let line = _this.createCPath(startleft, starttop, endleft, endtop);
        pathDom.setAttribute("data-start", startleft + "," + starttop);
        pathDom.setAttribute("data-end", endleft + "," + endtop);
        pathDom.setAttribute("d", line);
        pathDom.setAttribute("id", new Date().getTime());
        pathDom.setAttribute("stroke", "#eee");
        pathDom.setAttribute("stroke-width", "2");
        pathDom.setAttribute("marker-end", "url(#idArrow)");
        pathDom.setAttribute("fill", "none");
        document.getElementById("svg").appendChild(pathDom);
      };
    },
    changeLine(pathDom, startleft, starttop, endleft, endtop) {
      let line = this.createCPath(
        Number(startleft),
        Number(starttop),
        Number(endleft),
        Number(endtop)
      );
      pathDom.setAttribute("data-start", startleft + "," + starttop);
      pathDom.setAttribute("data-end", endleft + "," + endtop);
      pathDom.setAttribute("d", line);
    },
    queryLine(domArry, boxLeft, boxTop) {
      for (let i = 0; i < domArry.length; i++) {
        let startId = domArry[i].getAttribute("data-start")
          ? domArry[i].getAttribute("data-start").split(",")
          : [];
        let endId = domArry[i].getAttribute("data-end")
          ? domArry[i].getAttribute("data-end").split(",")
          : [];
        if (startId[0]) {
          for (let j = 0; j < startId.length; j++) {
            let lineDom = document.getElementById(startId[j]);
            let endleft = lineDom.getAttribute("data-end").split(",")[0];
            let endtop = lineDom.getAttribute("data-end").split(",")[1];
            let startleft = boxLeft + domArry[i].offsetLeft;
            let starttop = boxTop + domArry[i].offsetTop;
            this.changeLine(lineDom, startleft, starttop, endleft, endtop);
          }
        }
        if (endId[0]) {
          for (let j = 0; j < endId.length; j++) {
            let lineDom = document.getElementById(endId[j]);
            let startleft = lineDom.getAttribute("data-start").split(",")[0];
            let starttop = lineDom.getAttribute("data-start").split(",")[1];
            let endleft = boxLeft + domArry[i].offsetLeft;
            let endtop = boxTop + domArry[i].offsetTop;
            this.changeLine(lineDom, startleft, starttop, endleft, endtop);
          }
        }
      }
    },
    dragBlock() {
      let _this = this;
      document.onmousemove = function(ev) {
        //IE支持event，firefox不支持
        var oEvent = ev || event;
        let connetPoint = "";
        if (oEvent.target.children[0]) {
          if (oEvent.target.children[0].children) {
            connetPoint = oEvent.target.children[0].children;
          }
        }
        var scrollLeft =
          document.documentElement.scrollLeft || document.body.scrollLeft;
        var scrollTop =
          document.documentElement.scrollTop || document.body.scrollTop;
        var navtop = oEvent.clientY + scrollTop - _this.fTop;
        var navleft = oEvent.clientX + scrollLeft - _this.fLeft;
        if (navleft <= 40) {
          navleft = 40;
        }
        if (navtop <= 10) {
          navtop = 10;
        }
        if (navleft >= 940) {
          navleft = 940;
        }
        if (navtop >= 760) {
          navtop = 760;
        }
        _this.queryLine(connetPoint, navleft - 38, navtop - 10);
        _this.odiv.style.top = navtop - 10 + "px"; //Y
        _this.odiv.style.left = navleft - 40 + "px"; //X
      };
    }
  }
};
</script>
<style lang="scss" scoped>
.index {
  padding: 50px 0;
  display: flex;
  justify-content: center;
  .function-list {
    width: 100px;
    border: 1px solid #eee;
    margin: 0 50px 0 0;
    .fun-name {
      padding: 10px 0;
      cursor: pointer;
    }
  }
  .canvas-box {
    position: relative;
    border: 1px solid #eee;
    width: 1000px;
    height: 600px;
    .block {
      -moz-user-select: none; /*火狐*/
      -webkit-user-select: none; /*webkit浏览器*/
      -ms-user-select: none; /*IE10*/
      user-select: none;
      background: #fff;
      white-space: nowrap;
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      border: 1px solid #ddd;
      border-radius: 10px;
      width: 100px;
      height: 30px;
      text-align: center;
      line-height: 30px;
      font-size: 16px;
      .point-box {
        position: relative;
        visibility: hidden;
        width: 0;
        height: 0;
        left: 0;
        top: 0;
        .point {
          position: absolute;
          width: 3px;
          height: 3px;
          border-radius: 50%;
          border: 1px solid blue;
        }
        .point-top {
          top: -4px;
          right: -50px;
        }
        .point-bottom {
          bottom: -33px;
          right: -50px;
        }
      }
      &:hover {
        .point-box {
          visibility: visible;
        }
      }
    }
  }
}
</style>

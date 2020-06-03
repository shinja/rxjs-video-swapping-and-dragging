<template>
  <div>
    <div ref="dragTarget" class="video v-1">
      <video ref="video1" autoplay @loadeddata="initRxJs">
        <source
          src="https://www.w3schools.com/html/mov_bbb.mp4"
          type="video/mp4"
        />
        <source
          src="https://www.w3schools.com/html/mov_bbb.ogg"
          type="video/ogg"
        />
        Your browser does not support HTML video.
      </video>
      <div class="mask"></div>
    </div>

    <div ref="fixedTarget" class="video v-2">
      <video ref="video2" autoplay>
        <source
          src="https://storage.googleapis.com/webfundamentals-assets/videos/chrome.webm"
          type="video/webm"
        />
        <source
          src="https://storage.googleapis.com/webfundamentals-assets/videos/chrome.mp4"
          type="video/mp4"
        />
        <p>This browser does not support the video element.</p>
      </video>
    </div>
    <div>block3</div>
  </div>
</template>

<script>
import { fromEvent, empty } from "rxjs";
import {
  flatMap,
  takeUntil,
  withLatestFrom,
  timeoutWith,
  tap
} from "rxjs/operators";

export default {
  data() {
    return {
      dragging: null,
      swapping: null
    };
  },

  methods: {
    initRxJs() {
      this.dragging = !this.dragging && this.initDragging();
      this.swapping = !this.swapping && this.initSwap();
    },

    initDragging() {
      const dragTarget = this.$refs.dragTarget;

      const rect = dragTarget.getBoundingClientRect();

      const mousedown = fromEvent(dragTarget, "mousedown");
      const mousemove = fromEvent(window.document, "mousemove");
      const mouseup = fromEvent(window.document, "mouseup");

      return mousedown
        .pipe(
          flatMap(() => mousemove.pipe(takeUntil(mouseup))),
          withLatestFrom(mousedown, (move, down) => {
            return {
              x: this.validValue(
                move.clientX - down.offsetX,
                window.innerWidth - rect.width,
                0
              ),
              y: this.validValue(
                move.clientY - down.offsetY,
                window.innerHeight - rect.height,
                0
              )
            };
          })
        )
        .subscribe(pos => {
          dragTarget.style.left = `${pos.x}px`;
          dragTarget.style.top = `${pos.y}px`;
        });
    },

    initSwap() {
      const dragTarget = this.$refs.dragTarget;
      const video1 = this.$refs.video1;
      const video2 = this.$refs.video2;

      const mousedown = fromEvent(dragTarget, "mousedown");
      const mouseup = fromEvent(window.document, "mouseup");

      return mousedown
        .pipe(
          flatMap(() => mouseup.pipe(timeoutWith(200, empty()))),
          tap(e => console.log(e))
        )
        .subscribe(() => {
          this.swap(video1, video2);
        });
    },

    swap(x, y) {
      var temp = document.createElement("div");
      x.parentNode.insertBefore(temp, x);
      y.parentNode.insertBefore(x, y);
      temp.parentNode.insertBefore(y, temp);
      temp.parentNode.removeChild(temp);
    },

    validValue(value, max, min) {
      return Math.min(Math.max(value, min), max);
    }
  }
};
</script>

<style lang="scss" scoped>
body {
  font-family: sans-serif;
}

.video {
  position: relative;
  display: inline-block;
  line-height: 0; /* remove gap of video container */
}

.video.v-1 {
  position: absolute;
  z-index: 1;
  top: 0px;
  left: 0px;
}
.video.v-1:hover {
  cursor: grabbing;
}

.video.v-1 video {
  width: 320px;
}

.video.v-2 {
  width: 640px;
}
.video.v-2 video {
  width: 100%;
}

.video .mask {
  display: none;
  pointer-events: none;
}

.video:hover .mask {
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 2;
}
</style>

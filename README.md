### Altered to fix touch issues

Added createjs.opd.js

Added createjs.opd.min.js

All the changes are in EaselJS and relate only to touch events.

## Changes

# Touch.js

	/**
	 owendwyer - added || window.TouchEvent
	 **/
	Touch.isSupported = function() {
		return !!(('ontouchstart' in window) // iOS & Android
			|| window.TouchEvent
			|| (window.MSPointerEvent && window.navigator.msMaxTouchPoints > 0) // IE10
			|| (window.PointerEvent && window.navigator.maxTouchPoints > 0)); // IE11+
	};
  

	/**
	 owendwyer - added ||!stage.__touch
	 **/
	Touch.disable = function(stage) {
		if (!stage||!stage.__touch) { return; }
		if ('ontouchstart' in window) { Touch._IOS_disable(stage); }
		else if (window.PointerEvent || window.MSPointerEvent) { Touch._IE_disable(stage); }

		delete stage.__touch;
	};
  
  
	/**
	 owendwyer - added stage.enableDOMEvents(false) to remove DOM events when Touch is enabled
	 **/
	Touch._IE_enable = function(stage) {
		var canvas = stage.canvas;
		var f = stage.__touch.f = function(e) { Touch._IE_handleEvent(stage,e); };

		stage.enableDOMEvents(false);

		if (window.PointerEvent === undefined) {
			canvas.addEventListener("MSPointerDown", f, false);
			window.addEventListener("MSPointerMove", f, false);
			window.addEventListener("MSPointerUp", f, false);
			window.addEventListener("MSPointerCancel", f, false);
			if (stage.__touch.preventDefault) { canvas.style.msTouchAction = "none"; }
		} else {
			canvas.addEventListener("pointerdown", f, false);
			window.addEventListener("pointermove", f, false);
			window.addEventListener("pointerup", f, false);
			window.addEventListener("pointercancel", f, false);
			if (stage.__touch.preventDefault) { canvas.style.touchAction = "none"; }

		}
		stage.__touch.activeIDs = {};
	};
 
 
	/**
	 owendwyer - added stage.enableDOMEvents(true) to re-enable DOM events when Touch is disabled
	 **/
	Touch._IE_disable = function(stage) {
		var f = stage.__touch.f;

		stage.enableDOMEvents(true);

		if (window.PointerEvent === undefined) {
			window.removeEventListener("MSPointerMove", f, false);
			window.removeEventListener("MSPointerUp", f, false);
			window.removeEventListener("MSPointerCancel", f, false);
			if (stage.canvas) {
				stage.canvas.removeEventListener("MSPointerDown", f, false);
			}
		} else {
			window.removeEventListener("pointermove", f, false);
			window.removeEventListener("pointerup", f, false);
			window.removeEventListener("pointercancel", f, false);
			if (stage.canvas) {
				stage.canvas.removeEventListener("pointerdown", f, false);
			}
		}
	};
  
  
# Stage.js

	/**
	 owendwyer - this whole block was redone so that mousemove isn't removed
	 **/
	p.enableDOMEvents = function(enable) {
		if (enable == null) { enable = true; }
		var n, o, ls = this._eventListeners;
		if (!enable && ls) {
			for (n in ls) {
				o = ls[n];
				if(n!=="mousemove"){
					o.t.removeEventListener(n, o.f, false);
					delete this._eventListeners[n];
				}
			}
		} else if (enable && this.canvas) {
			var t = window.addEventListener ? window : document;
			var _this = this;
			if(this._eventListeners===undefined)ls=this._eventListeners={};
			if(ls["mousemove"]===undefined){
				ls["mousemove"] = {t:t, f:function(e) { _this._handleMouseMove(e)} };
				ls["mousemove"].t.addEventListener("mousemove",ls["mousemove"].f, false);
			}
			if(ls["mouseup"]===undefined){
				ls["mouseup"] = {t:t, f:function(e) { _this._handleMouseUp(e)} };
				ls["mouseup"].t.addEventListener("mouseup",ls["mouseup"].f, false);
			}
			if(ls["dblclick"]===undefined){
				ls["dblclick"] = {t:this.canvas, f:function(e) { _this._handleDoubleClick(e)} };
				ls["dblclick"].t.addEventListener("dblclick",ls["dblclick"].f, false);
			}
			if(ls["mousedown"]===undefined){
				ls["mousedown"] = {t:this.canvas, f:function(e) { _this._handleMouseDown(e)} };
				ls["mousedown"].t.addEventListener("mousedown",ls["mousedown"].f, false);
			}
		}
	};

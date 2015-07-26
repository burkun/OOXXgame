# OOXXgame
使用js实现的OOXX游戏，用到了蒙特卡洛模拟来进行人机博弈
可以修改代码自定义格子数量，颜色
<pre>
  var baseColor = "#999999";
  var blockSize = 130;
  var lineWidth = 4;
  var size = 3 //3*3的格子
</pre>
<script type="text/javascript">
window.onload=function(){
	var game_gui = function(canvas, gameObj, _refreshrate){
		if(gameObj && typeof gameObj["paint"] == "undefined"){
			throw new Error("game object do not implement paint method");
		}
		this.canvas = canvas;
		this.gameObj = gameObj || {paint:function(){}};
		this.refreshrate = _refreshrate || 10;
		this.timer = null;
		this.pause = function(){
			if(this.timer!=null){
				clearInterval(this.timer);
				this.timer = null;
			};
		};
		this.run = function(){
			this.pause();
			this_a = this;
			this.timer = setInterval(function(){
				//do someting else
				this_a.gameObj.paint(this_a.canvas);
			}, this.refreshrate);
		};
	};
	
	
	/**
	* 该对象负责绘画
	*/
	var gameObj = function(eventObj){
		var baseColor = "#999999";
		var blockSize = 130;
		var lineWidth = 4;
		var offsetX = 190;
		var offsetY = 100;
		var size = 3;
		var length = blockSize*size + lineWidth*size;
		var that = this;
		
		var getPointOnCanvas = function(x, y) {  
    		var bbox = eventObj.getBoundingClientRect();  
    		return { 
				x: x - bbox.left * (eventObj.width  / bbox.width) - offsetX,  
            	y: y - bbox.top  * (eventObj.height / bbox.height) - offsetY  
            };  
		};
		
		var getClickIndex = function(position){
			var indexX = Math.floor(position.x/blockSize);
			var indexY = Math.floor(position.y/blockSize);
			if(indexX<0 || indexY < 0 || indexX >= size || indexY >= size){
				return {x:-1, y:-1};
			}else{
				return {x: indexY, y:indexX};
			}
		};
		
		var getCenter = function(indexY, indexX){
			var centerX = offsetX + indexX * blockSize + (indexX+0.6)*lineWidth + blockSize/2;
			var centerY = offsetY + indexY * blockSize + (indexY+0.6)*lineWidth + blockSize/2;
			return {x:centerX, y:centerY};
		}
		
		var drawBoarder = function(ctx){
			//竖着四条线，横着四条线
			ctx.lineWidth = lineWidth;
			ctx.strokeStyle = baseColor;
			ctx.lineJoin="round";
			var idx = 0;
			for(idx = 0; idx < size+1; idx++){
				ctx.beginPath();
				var beginX = offsetX + idx*(blockSize+lineWidth);
				ctx.moveTo(beginX, offsetY);
				ctx.lineTo(beginX, offsetY + length);
				ctx.stroke();
			}
			for(idx = 0; idx < size+1; idx++){
				var beginY = offsetY + idx*(blockSize+lineWidth);
				ctx.moveTo(offsetX, beginY);
				ctx.lineTo(offsetX + length, beginY);
				ctx.stroke();
			}
		};
		
		
		var drawCircle = function(indexX, indexY, ctx, color){
			postion = getCenter(indexX, indexY);
			var col = color || "#91E333";
			ctx.lineWidth = 2;
			ctx.strokeStyle = col;
			
			ctx.beginPath();
            ctx.arc(postion.x, postion.y, blockSize*0.4 ,0,Math.PI * 2,true);
			//ctx.closePath();
			
			ctx.stroke();
			

		};
		
		var drawXX = function(indexX, indexY, ctx, color){
			postion = getCenter(indexX, indexY);
			offs = blockSize*0.25* 1.414;
			
			left_top = {x: postion.x - offs, y: postion.y - offs};
			right_bottom = {x: postion.x + offs, y: postion.y + offs};
			left_bottom = {x: postion.x - offs, y: postion.y + offs};
			right_top = {x: postion.x + offs, y: postion.y - offs};
			var col = color || "#E02EDF";
			ctx.lineWidth = 2;
			ctx.strokeStyle = col;
			
			ctx.beginPath();
			ctx.moveTo(left_top.x, left_top.y);
			ctx.lineTo(right_bottom.x , right_bottom.y);
			
			ctx.moveTo(left_bottom.x, left_bottom.y);
			ctx.lineTo(right_top.x , right_top.y);

			//ctx.closePath();
			
			ctx.stroke();

		};
		
		
		
		//-------------------------
		//数据逻辑部分
		this.logical = function(clone_obj){
			this.EMPTY = 0;
			this.XX = 1;
			this.OO = 10;
			this.DRAW = 30;
			this.curPlayer = this.XX;
			this.firstHand = this.XX;
			var _this = this;
			
			
			var data = null;
			if(clone_obj && typeof clone_obj["cloneData"] != "undefined"){
				data = clone_obj.cloneData();
			}else{
				data = function(){
					var aa=new Array();
					var idx, idxx;
					for(idx = 0; idx <size; idx++){
						aa[idx] = new Array();
						for(idxx = 0; idxx <size; idxx++){
							aa[idx][idxx] = _this.EMPTY;
						}
					}
					return aa;
				}();
			};
			
			this.reStart = function(){
				var idx, idxx;
				for(idx = 0; idx <size; idx++){
					for(idxx = 0; idxx <size; idxx++){
						data[idx][idxx] = _this.EMPTY;
					}
				}
				this.curPlayer = this.firstHand;
			};
			
			
			this.getEmptyBlocks = function(){
				var res = [];
				for(var i=0; i< size; i++){
					for(var j=0; j<size; j++){
						if(data[i][j] == _this.EMPTY){
							res.push([i,j]);
						}
					}
				}
				return res;
			};
			
			
			this.switchPlayer = function(player){
				if(player == _this.XX){
					this.curPlayer = _this.OO;
					return this.OO;
				}else{
					this.curPlayer = _this.XX;
					return _this.XX;	
				}
			};
			
			this.cloneData = function(){
				var aa=new Array();
				var idx, idxx;
				for(idx = 0; idx <size; idx++){
					aa[idx] = new Array();
					for(idxx = 0; idxx <size; idxx++){
						aa[idx][idxx] = data[idx][idxx];
					}
				}
				return aa
			};
			
			this.getData= function(indexX, indexY){
				if(indexX > -1 && indexX < size && indexY > -1 && indexY < size){
					return data[indexX][indexY];
				}
			};
			
			this.move = function(indexX, indexY, player){
				if(indexX > -1 && indexX < size && indexY > -1 && indexY < size){
					return data[indexX][indexY] = player;
				}
			};
			
			this.isWin = function(){
				var idx, idy;
				sum1 = 0;
				sum3 = 0;
				for(idx =0 ; idx < size; idx++){
					sum3 += data[idx][idx];
					for(idy = 0; idy < size; idy++){
						sum1 += data[idx][idy];
					}
					if(sum1 == _this.XX * size || sum3 == _this.XX * size){
						return _this.XX;
					}else if(sum1 == _this.OO * size || sum3 == _this.OO * size){
						return _this.OO;
					}
					sum1 = 0;
				}
				sum3 = 0;
				for(idx =0 ; idx < size; idx++){
					sum3 += data[idx][size-idx-1];
					for(idy = 0; idy < size; idy++){
						sum1 += data[idy][idx];
					}
					if(sum1 == _this.XX * size || sum3 == _this.XX * size){
						return _this.XX;
					}else if(sum1 == _this.OO * size || sum3 == _this.OO * size){
						return _this.OO;
					}
					sum1 = 0;
				}
				if (_this.getEmptyBlocks().length == 0){
					return _this.DRAW
				}
				return null;
			}
			
		};


		this.copyLogical = function(obj){
			return new this.logical(obj);
		}
		
		//-------------------------多次模拟
		var mc_trial = function(){
			
			var _this = this;    
			var MCMATCH = 1.0  
			var MCOTHER = 1.0
		
			
			var randomChose = function(arr){
				var len = arr.length;
				var index = parseInt(Math.random() * len);
				return arr[index];
			};
			
			
			var run_once = function(manager_clone){
				var cplayer = manager_clone.curPlayer;
				while(manager_clone.isWin() == null){
						empty_blocks =manager_clone.getEmptyBlocks();
						posIndex = randomChose(empty_blocks);
						manager_clone.move(posIndex[0], posIndex[1], cplayer);
						cplayer = manager_clone.switchPlayer(cplayer);
				}
			};
			
			var mc_update_scores = function(manager_clone, player, scores){
				var res = manager_clone.isWin();
				var factor = 1;
				if(res == manager_clone.DRAW) return;
				if(res != null){
					if(res != player){
						factor = -1;
					}
					for(var i = 0 ; i < size; i++){
						for(var j=0; j< size; j++){
							var temp = manager_clone.getData(i,j);
							if(temp == player){
								scores[i][j] += factor * MCMATCH;
							}else if(temp == manager_clone.EMPTY){
								scores[i][j] += 0;
							}else{
								scores[i][j] += -1*factor*MCOTHER;
							}
						}
					}
				}
			};
			
			var get_best_move = function(manager_clone, scores){
				var empty_blocks = manager_clone.getEmptyBlocks();
				var maxPos = [];
				var max_score = -99999999;
				for(var i =0; i< empty_blocks.length; i++){
					var it = empty_blocks[i];
					if(max_score < scores[it[0]][it[1]]){
						maxPos = it;
						max_score = scores[it[0]][it[1]];
					}
				}
				return maxPos;
			};
			
			this.mc_move = function(manager, player, trails){
				var scores = function(){
					var idx, idxx;
					var aa = [];
					for(idx = 0; idx <size; idx++){
						aa[idx] = [];
						for(idxx = 0; idxx <size; idxx++){
							aa[idx][idxx] = 0;
						}
					}
					return aa;
				}();
				for(var i = 0; i< trails; i++){
					var manager_clone = that.copyLogical(manager);
					run_once(manager_clone);
					mc_update_scores(manager_clone, player, scores);
				}
				return get_best_move(manager, scores);
			};
			
			
		}
		//-------------------------
		var manager  = new this.logical();
		var mc_t = new mc_trial();
		this.paint = function(ctx){
			ctx.clearRect(0,0,800,600);
			var idx, idxx;
			for(idx = 0; idx <size; idx++){
				for(idxx = 0; idxx <size; idxx++){
					if(manager.getData(idx,idxx) == manager.XX){
						console.log("XX1:"+ ctx.strokeStyle);
						drawXX(idx,idxx, ctx);
						console.log("XX2:"+ ctx.strokeStyle);
					}else if(manager.getData(idx,idxx) == manager.OO){
						console.log("OO1:"+ ctx.strokeStyle);
						drawCircle(idx,idxx, ctx);
						console.log("OO2:"+ ctx.strokeStyle);
					}
				}
			}
			
			drawBoarder(ctx);
		};
		
		eventObj.onclick = function(evt){
			
			var pos = getClickIndex(getPointOnCanvas(evt.clientX, evt.clientY));
			if(pos.x > -1 && pos.y > -1 && manager.getData(pos.x, pos.y) == manager.EMPTY && manager.isWin() == null){
				manager.move(pos.x, pos.y, manager.curPlayer);
				manager.switchPlayer(manager.curPlayer);
				var res = manager.isWin();
				if(res == null){
					var next = mc_t.mc_move(manager, manager.curPlayer, 20000);
					manager.move(next[0], next[1], manager.curPlayer);
					var new_res = manager.isWin();
					if(new_res){
						if(new_res == manager.DRAW){
							alert("平局！");
						}else if(new_res == manager.XX){
							alert("XX赢！");
						}else{
							alert("OO赢！");
						}
						manager.reStart();
					}else{
						manager.switchPlayer(manager.curPlayer);
					}
				}else{
					if(res == manager.DRAW){
						alert("平局！");
					}else if(res == manager.XX){
						alert("XX赢！");
					}else{
						alert("OO赢！");
					}
					manager.reStart();
				}
			}
			
		}
	};
	
	
	var can = document.getElementById("content").getContext("2d");
	var gui = new game_gui(can,new gameObj(document.getElementById("content")) , 100);
	gui.run();

}


</script>

<div id="base" style="margin:0 auto; width:800px; margin-top:40px;">
	<canvas id="content" width="800px" height="600px" style="border:1px solid #E82D30; background-color:#2F2929"></canvas>
</div>

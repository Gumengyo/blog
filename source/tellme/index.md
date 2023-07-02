---
title: 留言板
date: 2023-06-28 23:06:06
comment: true
---

<style>
body, div, h1,h2, form, fieldset, footer,p {
	margin: 0; padding: 0; border: 0; outline: none;
}
body {
	color: #7c7873;
	font-family: 'YanoneKaffeesatzRegular';
	background-color: #D7D7D7;
}
p {text-shadow:0 1px 0 #fff;}
#wrap {width:530px; margin:20px auto 0; }
h1 {margin-bottom:20px; text-align:center;font-size:48px; text-shadow:0 1px 0 #ede8d9; }
	#form_wrap { overflow:hidden; height:446px; position:relative; top:0px;
		-webkit-transition: all 1s ease-in-out .3s;
		-moz-transition: all 1s ease-in-out .3s;
		-o-transition: all 1s ease-in-out .3s;
		transition: all 1s ease-in-out .3s;}
	#form_wrap:before {content:"";
		position:absolute;
		bottom:128px;left:0px;
		background:url('https://101.43.39.125/HexoFiles/images/before.png');
		width:530px;height: 316px;}
	#form_wrap:after {content:"";position:absolute;
		bottom:0px;left:0;
		background:url('https://101.43.39.125/HexoFiles/images/after.png');
		width:530px;height: 260px; }
	#form_wrap.hide:after, #form_wrap.hide:before {display:none; }
	#form_wrap:hover {height:806px;top:-30px;}
	form {background:#f7f2ec url('https://101.43.39.125/HexoFiles/images/letter_bg.png'); 
		position:relative;top:200px;overflow:hidden;
		height:200px;width:400px;margin:0px auto;padding:20px; 
		border: 1px solid #fff;
		border-radius: 3px; 
		-moz-border-radius: 3px; -webkit-border-radius: 3px;
		box-shadow: 0px 0px 3px #9d9d9d, inset 0px 0px 27px #fff;
		-moz-box-shadow: 0px 0px 3px #9d9d9d, inset 0px 0px 14px #fff;
		-webkit-box-shadow: 0px 0px 3px #9d9d9d, inset 0px 0px 27px #fff;
		-webkit-transition: all 1s ease-in-out .3s;
		-moz-transition: all 1s ease-in-out .3s;
		-o-transition: all 1s ease-in-out .3s;
		transition: all 1s ease-in-out .3s;}
		#form_wrap:hover form {height:530px;}
		label {
			margin: 11px 20px 0 0; 
			font-size: 16px; color: #b3aba1;
			text-transform: uppercase; 
			text-shadow: 0px 1px 0px #fff;
		}

		#form_wrap input[type=submit] {	
			position:relative;font-family: 'YanoneKaffeesatzRegular'; 	
			font-size:24px; color: #7c7873;text-shadow:0 1px 0 #fff;	
			width:100%; text-align:center;opacity:0;	
			background:none;	
			cursor: pointer;	
			-moz-border-radius: 3px; -webkit-border-radius: 3px; border-radius: 3px; 	
			-webkit-transition: opacity .6s ease-in-out 0s;	
			-moz-transition: opacity .6s ease-in-out 0s;	
			-o-transition: opacity .6s ease-in-out 0s;	
			transition: opacity .6s ease-in-out 0s;	
		}
		#form_wrap:hover input[type=submit] {z-index:1;opacity:1;	
			-webkit-transition: opacity .5s ease-in-out 1.3s;	
			-moz-transition: opacity .5s ease-in-out 1.3s;	
			-o-transition: opacity .5s ease-in-out 1.3s;	
			transition: opacity .5s ease-in-out 1.3s;}
			#form_wrap:hover input:hover[type=submit] {color:#435c70;}
  .top-banner {
    background-color: #555;
  }
</style>
<div style="margin-top: -10px">
	<div id="wrap">
		<div id='form_wrap'>
			<form>
				<img src='https://cdn.jishuqin.cn/blog/violet.jpg' height=245px/>
                 <p> Hi ，</p>
				<label for="email"> </label>
		欢迎来到我的博客留言板！我期待着与你一起交流，分享思考和感悟。在这里，留下你的足迹，让我们一起交流，探索更多美好的可能！
		</div>
	</div>
</div>

<script src="http://www.jq22.com/jquery/1.7.2/jquery.min.js"></script>

---

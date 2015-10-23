 /*
	$Id: A*K giftcollector.js,v 1.39 2013-09-23 14:33:16 A*K Exp $
	Collect all your gifts from the ZMC (and not more)
	Author: A*K, Team Spockholm
	Todo:
	Spocklet complete
*/


javascript:(function (){
	var rev=/,v \d+\.(\d+)\s201/.exec("$Id: giftcollector.js,v 1.39 2013-09-23 14:33:16 eike Exp $")[1],
	version=' Gift-Collector v1.'+rev;
	var spocklet='giftcollect';
	var debug = false;
	var run = false;
	var stats = { lastrun: 0,count:0 };
	var more_items = {};
	var storage;
	var mafia = { start_att:0, cur_att:0, start_def:0, cur_def:0,start_attsk:0, cur_attsk:0, start_defsk:0, cur_defsk:0 };
	//setup images
	var mafia_attack = imgurl('icon_mafia_attack_22x16_01.gif','22','16','Mafia Attack Strength');
	var mafia_defense = imgurl('icon_mafia_defense_22x16_01.gif','22','16','Mafia Defense Strength');
	var mafia_attack = imgurl('icon_mafia_attack_22x16_01.gif','22','16','Mafia Attack Strength');
	var mafia_defense = imgurl('icon_mafia_defense_22x16_01.gif','22','16','Mafia Defense Strength');
	var parameters = get_params();

	try {
		if (localStorage.getItem) {
			storage = localStorage;
		}
		else if (window.localStorage.getItem) {
			storage = window.localStorage;
		}
	}
	catch(storagefail) {}

	if (navigator.appName == 'Microsoft Internet Explorer') {
		alert('You are using Internet Explorer, this bookmarklet will not work.\nUse Firefox or Chrome instead.');
		return;
	}
	if (/m.mafiawars.com/.test(document.location)) {
		window.location.href = document.location+'?iframe=1';
	}
	else if (/apps.facebook.com.inthemafia/.test(document.location)) {
		//Credits to Christopher(?) for this new fix
		for (var i = 0; i < document.forms.length; i++) {
			if (/canvas_iframe_post/.test(document.forms[i].id) && document.forms[i].target == "mafiawars") {
				document.forms[i].target = '';
				document.forms[i].submit();
				return;
			}
		}
	}
	else if (document.getElementById('some_mwiframe')) {
		// new mafiawars.com iframe
		window.location.href = document.getElementById('some_mwiframe').src;
		return;
	}
	else {
		document.body.parentNode.style.overflowY = "scroll";
		document.body.style.overflowX = "auto";
		document.body.style.overflowY = "auto";
		try {
			document.evaluate('//div[@id="mw_city_wrapper"]/div', document, null, XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, null).snapshotItem(0).style.margin = "auto";
			if (typeof FB != 'undefined') {
				FB.CanvasClient.stopTimerToSizeToContent;
				window.clearInterval(FB.CanvasClient._timer);
				FB.CanvasClient._timer = -1;
			}
			document.getElementById('snapi_zbar').parentNode.parentNode.removeChild(document.getElementById('snapi_zbar').parentNode);

		} catch (fberr) {}
		//Revolving Revolver of Death from Arun, http://arun.keen-computing.co.uk/?page_id=33
		$('#LoadingBackground').hide();
		$('#LoadingOverlay').hide();
		$('#LoadingRefresh').hide();
	}
	//end unframe code

	//hack to kill the resizer calls
	iframeResizePipe = function() {};

	try {
		var saved_stats = JSON.parse(window.localStorage.getItem(spocklet+'_giftcount_'+User.trackId));
		if ((new Date(saved_stats.lastrun*1000)).getDay()!=(new Date()).getDay()) { saved_stats.count=0; saved_stats.lastrun=unix_timestamp();}
		if (saved_stats.count && saved_stats.lastrun) {
			stats=saved_stats;
		}
	} catch (dannhaltnicht) { }
	
	try {
		var saved_more_items = JSON.parse(window.localStorage.getItem(spocklet+'_items_'+User.trackId));
		more_items = saved_more_items;
	} catch (dannhaltnicht) { }
		
	var http = 'http://';
	if (/https/.test(document.location)) {
		http = 'https://';
	}
	var preurl = http+'facebook-aws.mafiawars.zynga.com/mwfb/remote/html_server.php?';

	//setup the initial html
	var style = '<style type="text/css">'+
		'.messages {border: 1px solid #888888; margin-bottom: 5px; -moz-border-radius: 5px; border-radius: 5px; -webkit-border-radius: 5px;}'+
		'.messages img{margin:0 3px;vertical-align:middle}'+
		'.messages input {border: 1px solid #888888; -moz-border-radius: 3px; border-radius: 3px; -webkit-border-radius: 2px;padding 0;background: #000; color: #FFF; width: 32px;}'+
	'</style>';

	var logo = '<a href="http://www.spockholm.com/mafia/testing.php#" target="_blank"><img src="http://www.mafia-tools.com/uploads/banner-spockholm-mafia-tools.png#" alt="Spockholm Mafia Tools" title="Spockholm Mafia Tools" width="425" height="60" style="margin-bottom: 5px;" /></a>';

	var spocklet_html =
	'<div id="'+spocklet+'_main">'+
		style+
		'<table class="messages">'+
		'<tr><td colspan="3" align="center" style="text-align:center;">'+logo+'<br />'+
			'<a href="http://www.spockholm.com/mafia/donate.php#" id="'+spocklet+'_donate" class="sexy_button_new short white" target="_blank" title="Like what we do? Donate to Team Spockholm" alt="Like what we do? Donate to Team Spockholm"><span><span>Donate</span></span></a>'+
//			'&nbsp;<a href="http://www.spockholm.com/mafia/help.php#" id="'+spocklet+'_help" class="sexy_button_new short" target="_blank" title="Get help" alt="Get help"><span><span>Help</span></span></a>'+
			'&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="good">'+version+'</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'+
			'&nbsp;<a href="#" id="'+spocklet+'_close" class="sexy_button_new short red" title="Close" alt="Close"><span><span>Close</span></span></a>'+
		'</td></tr>'+
		'<tr><td colspan="3"><hr /></td></tr>'+
		'<tr><td colspan="3"><span id="'+spocklet+'_stats"></span></td></tr>'+
		'<tr><td colspan="3"><span id="'+spocklet+'_gifts">Loading...</span></td></tr>'+
		'<tr><td colspan="3"><span id="'+spocklet+'_loot"></span></td></tr>'+
		'<tr><td colspan="3"><span id="'+spocklet+'_sentback"></span></td></tr>'+
		'<tr><td colspan="3">Showing <input id="'+spocklet+'_loglines" type="number" value="20" style="width:50px;" /> lines of log:<br /><span id="'+spocklet+'_log"></span></td></tr>'+
		'</table>'+
	'</div>';

	
	function unix_timestamp() {
		return parseInt(new Date().getTime().toString().substring(0, 10));
	}
	function cmp(v1,v2){
		return (v1>v2?-1:(v1<v2?1:0));
	}
	
	function create_div() {
		//setup the spockdiv
		if ($('#'+spocklet+'_main').length == 0) {
			$('#inner_page').before(spocklet_html);
		}
		else {
			$('#'+spocklet+'_log').html();
		}
	}
	create_div();
	read_zmc(display_gifts);
	
	var gifts=[]; 
	var giftcats={}; // list of gifts available
	var gifttypes={
		'Exotic Animal Feed':{id:4604,cat:1},
		'Special Part':{id:2319 ,cat:1},
		'Brief Case Gun':{id:112 ,cat:4},
		'Good Word':{id:113,cat:4},
		'Toe Jammers':{id:114,cat:4},
		'Blue Mystery Bag':{id:527 ,cat:1},
		'Red Mystery Bag':{id:736 ,cat:1},
		'Special Part':{id:2319 ,cat:1},
		'Secret Drop':{id:4400 ,cat:1},
		'Exotic Animal Feed':{id:4604 ,cat:1},
		'Antivenom':{id:8327 ,cat:1},
		'Union Worker':{id:11051 ,cat:1},
		'Carpenter Nails':{id:11052 ,cat:1},
		'Lath Strips':{id:11053 ,cat:1},
		'Iron Cast':{id:11054 ,cat:1},
		'Douglas Fir Beams':{id:11055 ,cat:1},
		'Set of Hidden Charges':{id:2610 ,cat:1},
		'Severed Pinky':{id:2611 ,cat:1},
		'Cooked Book':{id:2614 ,cat:1},
		'Italian Hardwood':{id:2600 ,cat:1},
		'Marble Slab':{id:2601 ,cat:1},
		'Stone Column':{id:2602 ,cat:1},
		'Set of Terracotta Tiles':{id:2603 ,cat:1},
		'Mystery Boost':{id:1 ,cat:8},
		//'':{id: ,cat:1},
		//'':{id: ,cat:1},
		//'':{id: ,cat:1},
	}; // list of gift types
	var logs=[];
	var loot={}; // collected gifts
	var sentback = 0;
	for (var i in saved_more_items) {
		gifttypes[i]=saved_more_items[i];
	}

	// get gifts from the zmc
	function read_zmc(handler){
		request('xw_controller=messageCenter&xw_action=view&xw_city=1&tmp=&cb=&xw_person='+User.id.substr(2)+'&mwcom=1&xw_client_id=8',function(msg){
			$msg=$(msg);
			window.eike=msg;
			$msg.find('.gift,.help,.energy_pack,.mafia_invite').each(function(){
				
				var data={name:''};
				var m;
				try {
					data.link=/html_server.php.([^\"]*)\"/.exec($(this).find('.state_accept>a:first').attr('onclick'))[1];
				} catch(e) {}
				try {
					data.ignore=/html_server.php.([^\"]*)\'/.exec($(this).find('a.ignore:first').attr('onclick'))[1];
				} catch(e) {}
				
				data.text=$(this).find('p[id*=zmc_bodytext]').text().trim();
				data.img=$(this).find('img[id*=zmc_itempic]').attr('src');
				if (m=/sent you a?n?\s?(.*) to /.exec(data.text)) {
					data.name=m[1];
				} else if (m=/sent you a?n?\s?([^\.]*)\./.exec(data.text)) {
					data.name=m[1];
				} else if (m=/I need your help to complete my mission/.exec(data.text)) {
					data.name='Mission Crew';
				} else if (m=/been selected to join/.exec(data.text)) {
					data.name='City Crew';
				} else if (m=/wants to join forces/.exec(data.text)) {
					data.name='Mafia Invite';
				} else if (m=/Book your tickets to London/.exec(data.text)) {
					data.name='London Invite';
				} else if (m=/THE MOB DOCTOR/.exec(data.text)) {
					data.name='MOB DOCTOR Bag';
				} else if (m=/Savanna by storm/.exec(data.text)) {
					data.name='Head Hunter';
				}
				try {
					data.sendback=/sendGiftbackRequest\(([^\)]*)\)/.exec($(this).find('.state_accept>a:first').attr('onclick'))[1];
				} catch(e) {}
				if (m=/^(.*) in Mafia Wars/.exec(data.name)) {
					data.name=m[1];
				}
				if(data.link && data.text && data.name) {
					if(!giftcats[data.name]) { giftcats[data.name]=[]; }
					giftcats[data.name].push(data);
				} else {
					//console.log(data);
				}
			});
			handler();
		});
	}
	
	// show data and form
	// improved version
	function display_gifts(){
		var options='',html='<table cellpadding=0 cellspacing=0>';
		for(var cat in giftcats) {
			
			html+='<tr>'+
			'<td><span class="good '+spocklet+'_count">'+giftcats[cat].length+'</span> &times&nbsp;</td>'+
			'<td><b>'+cat+'</b></td>'+
			'<td>&nbsp;<a href="#" class="'+spocklet+'_accept">Accept</a>&nbsp;</td>'+
			'<td><input type="number" class="'+spocklet+'_acceptcount" style="width:100px;" value="0" max='+giftcats[cat].length+' min=0 />.</td>'+
			'<td>&nbsp;<a href="#" class="'+spocklet+'_ignore">Ignore</a>&nbsp;</td>'+
			'<td><input type="number" class="'+spocklet+'_ignorecount" style="width:100px;" value="0" max='+giftcats[cat].length+' min=0 />.</td>'+
			'<td>Keep other <span class="'+spocklet+'_remainingcount">'+giftcats[cat].length+'</span>.</td></tr>';
		}
		html+='</table><input type="checkbox" id="'+spocklet+'_sendback" unchecked />Send gifts back</label> - <a href="#" id="'+spocklet+'_acceptall">Accept all [<span id="'+spocklet+'_accallcount">0</span>]</a> - <a href="#" id="'+spocklet+'_ignoreall">Ignore all [<span id="'+spocklet+'_igallcount">0</span>]</a> - <a href="#" id="'+spocklet+'_savecookie">Save as default</a> - <a href="#" id="'+spocklet+'_loadcookie">Load defaults</a><br /><a href="#" id="'+spocklet+'_start" class="sexy_button_new short green" title="Start" alt="Start"><span><span>Start</span></span></a>';
		
		$('#'+spocklet+'_gifts').html(html);
		$('#'+spocklet+'_stats').html('Already accepted today: '+stats.count);
		
		$('#'+spocklet+'_start').click(function(){

				run = true;
				collect_gift();
				setTimeout(collect_gift,50);
				setTimeout(collect_gift,100);
				setTimeout(collect_gift,150);
				setTimeout(collect_gift,200);
				setTimeout(collect_gift,250);
				setTimeout(collect_gift,300);
				setTimeout(collect_gift,350);
				setTimeout(collect_gift,400);
				setTimeout(collect_gift,450);
			return false;
		});

		$('#'+spocklet+'_savecookie').click(function(){
			var settings={ ignoreall:{},acceptall:{},ignore:{},accept:{} };
			$('#'+spocklet+'_gifts tr').each(function(){
				var name=$(this).find('b').text();
				var num=parseInt($(this).find('.'+spocklet+'_count').text());
				var accept=parseInt($(this).find('.'+spocklet+'_acceptcount').val());
				var ignore=parseInt($(this).find('.'+spocklet+'_ignorecount').val());
				if(accept==num) { settings.acceptall[name]=1; } 
				else if (ignore==num) { settings.ignoreall[name]=1; } 
				else {
					if (ignore>0)  { settings.ignore[name]=ignore; } 
					if (accept>0)  { settings.accept[name]=accept; } 
				}
			});
//			console.log(settings);
			storage.setItem(spocklet+'_settings',JSON.stringify(settings));
		});

		$('#'+spocklet+'_loadcookie').click(function(){
			try {
				var settings=JSON.parse(storage.getItem(spocklet+'_settings'));
				$('#'+spocklet+'_gifts tr').each(function(){
					var name=$(this).find('b').text();
					var num=parseInt($(this).find('.'+spocklet+'_count').text());
					if(settings.acceptall[name]) {
						$(this).find('.'+spocklet+'_accept').trigger('click');
					}
					if(settings.ignoreall[name]) {
						$(this).find('.'+spocklet+'_ignore').trigger('click');
					}
					if(settings.accept[name]) {
						$(this).find('.'+spocklet+'_acceptcount').val(min(num,settings.accept[name])).trigger('change');
					}
					if(settings.ignore[name]) {
						$(this).find('.'+spocklet+'_ignorecount').val(min(num,settings.ignore[name])).trigger('change');
					}
					
				});
			} catch(e) {}
		});

		$('#'+spocklet+'_acceptall').click(function(){
			$('#'+spocklet+'_gifts').find('a.'+spocklet+'_accept').trigger('click');
		});
		
		$('#'+spocklet+'_ignoreall').click(function(){
			$('#'+spocklet+'_gifts').find('a.'+spocklet+'_ignore').trigger('click');
		});
		
		$('#'+spocklet+'_gifts').find('a.'+spocklet+'_accept').click(function(){
			var c=0,$tr=$(this).parent().parent();
			var count=parseInt($tr.find('.'+spocklet+'_count').text());

			$tr.find('input.'+spocklet+'_acceptcount').val(count);
			$tr.find('input.'+spocklet+'_ignorecount').val(0);
			$tr.find('input.'+spocklet+'_acceptcount').trigger('change');
			$('.'+spocklet+'_acceptcount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_accallcount').text(c);
			c=0;
			$('.'+spocklet+'_ignorecount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_igallcount').text(c);
			return false;
		});
		$('#'+spocklet+'_gifts').find('a.'+spocklet+'_ignore').click(function(){
			var c=0,$tr=$(this).parent().parent();
			var count=parseInt($tr.find('.'+spocklet+'_count').text());
			
			$tr.find('input.'+spocklet+'_ignorecount').val(count);
			$tr.find('input.'+spocklet+'_acceptcount').val(0);
			$tr.find('input.'+spocklet+'_ignorecount').trigger('change');
			$('.'+spocklet+'_acceptcount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_accallcount').text(c);
			c=0;
			$('.'+spocklet+'_ignorecount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_igallcount').text(c);
			return false;
		});
		$('#'+spocklet+'_gifts').find('input.'+spocklet+'_acceptcount,input.'+spocklet+'_ignorecount').change(function(){
			var c=0,$tr=$(this).parent().parent();
			var count=parseInt($tr.find('.'+spocklet+'_count').text());
			var ignorecount=parseInt($tr.find('input.'+spocklet+'_ignorecount').val());
			var acceptcount=parseInt($tr.find('input.'+spocklet+'_acceptcount').val());

			//console.log(count+' '+ignorecount+' '+acceptcount);
			$tr.find('input.'+spocklet+'_ignorecount').attr('max',count-acceptcount);
			$tr.find('input.'+spocklet+'_acceptcount').attr('max',count-ignorecount);
			$tr.find('.'+spocklet+'_remainingcount').text(count-ignorecount-acceptcount);

			$tr.find('a.'+spocklet+'_accept').css('color',((count==acceptcount)?'red':''));
			$tr.find('a.'+spocklet+'_ignore').css('color',((count==ignorecount)?'red':''));
			$('.'+spocklet+'_acceptcount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_accallcount').text(c);
			c=0;
			$('.'+spocklet+'_ignorecount').each(function(){ c+=parseInt(this.value); });
			$('#'+spocklet+'_igallcount').text(c);
			
			return false;
		}).click(function(){ $(this).trigger('change'); }); // both click and change
		
		if(parameters.auto) {
			$('#'+spocklet+'_loadcookie').trigger('click');
			$('#'+spocklet+'_start').trigger('click');
		}
		
	}

	// recursive function to process all gifts
	function collect_gift(){
		var $tr=$('#'+spocklet+'_gifts tr:first ');
		if(($tr.length>0) && run) {
			var $acceptcount=$tr.find('input.'+spocklet+'_acceptcount');
			var acceptcount=parseInt($acceptcount.val());
			var $ignorecount=$tr.find('input.'+spocklet+'_ignorecount');
			var ignorecount=parseInt($ignorecount.val());
			var gift=$tr.find('b').text();
			
			if (!(acceptcount>0)) {
				if(ignorecount>0) {
					ignoregift(gift,collect_gift);
				} else {
					//$('#giftcollect_gifts tr:first').fadeOut('slow',function(){ $(this).remove();collect_gift();})
					$('#giftcollect_gifts tr:first').remove();
					collect_gift();
				}
				return;
			}
			if (giftcats[gift].length>0) {
				var giftdata=giftcats[gift].pop();
				request(giftdata.link,function(msg){
					var data;
					try {
						data=JSON.parse(msg);
					} catch(e) { log('Undefined Error, trying next gift'); }
					if (data) {
						if(data.is_success) {
							add_loot(data.success_message,data.item_image_path || giftdata.img);
							// update counts
							acceptcount--;
							$acceptcount.val(acceptcount);

							stats.count++; stats.lastrun=unix_timestamp();
							window.localStorage.setItem(spocklet+'_giftcount_'+User.trackId,JSON.stringify(stats));
							mafia.cur_att=parseInt(data.fightbar.group_atk);
							mafia.cur_def=parseInt(data.fightbar.group_def);
							mafia.cur_attsk=parseInt(data.fightbar.skill_atk);
							mafia.cur_defsk=parseInt(data.fightbar.skill_def);
							if(mafia.start_def==0) { mafia.start_def=mafia.cur_def; mafia.start_att=mafia.cur_att;mafia.start_defsk=mafia.cur_defsk; mafia.start_attsk=mafia.cur_attsk;}
							var mafstr="<br />";
							
							if (mafia.cur_att!=mafia.start_att) { mafstr+=mafia_attack +' '+commas(mafia.cur_att)+' (<span class="'+((mafia.cur_att-mafia.start_att) >= 0?'good':'bad')+'">'+((mafia.cur_att-mafia.start_att)>0?'+':'')+commas(mafia.cur_att-mafia.start_att)+'</span>) '; }
							if (mafia.cur_def!=mafia.start_def) { mafstr+=mafia_defense+' '+commas(mafia.cur_def)+' (<span class="'+((mafia.cur_def-mafia.start_def) >= 0?'good':'bad')+'">'+((mafia.cur_def-mafia.start_def)>0?'+':'')+commas(mafia.cur_def-mafia.start_def)+'</span>) '; }
							if (mafia.cur_attsk!=mafia.start_attsk) { mafstr+='<span class="attack">'+commas(mafia.cur_attsk)+' (<span class="'+((mafia.cur_attsk-mafia.start_attsk) >= 0?'good':'bad')+'">'+((mafia.cur_attsk-mafia.start_attsk)>0?'+':'')+commas(mafia.cur_attsk-mafia.start_attsk)+'</span>)</span>'; }
							if (mafia.cur_defsk!=mafia.start_defsk) { mafstr+='<span class="defense">'+commas(mafia.cur_defsk)+' (<span class="'+((mafia.cur_defsk-mafia.start_defsk) >= 0?'good':'bad')+'">'+((mafia.cur_defsk-mafia.start_defsk)>0?'+':'')+commas(mafia.cur_defsk-mafia.start_defsk)+'</span>)</span>'; }
							
							$('#'+spocklet+'_stats').html('Already accepted today: '+stats.count+mafstr);
							
						} else {
							if (data.fail_message) {
								log(data.fail_message);
								if (data.fail_message.indexOf && (data.fail_message.indexOf('You cannot accept any more Free Gifts today.')!=-1)) {
									run = false;
									log('<span class="bad">Daily Gift limit reached!</span>');
									stats.lastrun=unix_timestamp(); 
									window.localStorage.setItem(spocklet+'_giftcount_'+User.trackId,JSON.stringify(stats));
								}
							}
							
						}
					} 
					if($('#'+spocklet+'_sendback').is(':checked')) {
						if(giftdata.sendback){
							
							var url = 'xw_controller=requests&xw_action=postSend&xw_city=7&tmp=&cb=&xw_person='+User.id.substr(2)+'&mwcom=1';
							var parts = JSON.parse('[' + giftdata.sendback + ']');
							
							url = url + '&rid=' + (parseInt(Math.random()*1000000000));
							url = url + '&data=' + parts[2];
							url = url + '&to=' + parts[0]; 

							request(url,function(msg){
								try {
									var $msg=$((JSON.parse(msg)).data.reqsentmsg);
									log('Sending back gift: '+$msg.find('div:first').text());
									sentback++;
									$('#'+spocklet+'_sentback').html('<b>Gifts sent back: '+sentback+'</b>');
									try { update_game_ui_gc(JSON.parse(msg)); } catch(ignore) {}
								} catch(e) {}
								collect_gift();
							},function(){
								log('Unknown error, retrying...');
								collect_gift();
							});
						} else {
							log('No Thank You for this link available');
							collect_gift();
						}
					} else {
						// no sending back
						collect_gift();
					}
				},function(){
					// error, retry
					log('Request error, trying next gift');
					collect_gift();
				});
			} else {
				log('There was an error with the request');
				$('#giftcollect_gifts tr:first').remove();
				collect_gift();
			}
		} else {
			if (!run) {
				log('Stopped');
			} else {
				log('Stopped, all gifts processed.');
			}
			run = false;
		}
		
		
	};
	function ignoregift(giftname,callback){
		var giftdata=giftcats[giftname].shift();

		var $tr=$('#'+spocklet+'_gifts tr:first ');
		var $ignorecount=$tr.find('input.'+spocklet+'_ignorecount');
		var ignorecount=parseInt($ignorecount.val());
		$ignorecount.val(ignorecount-1);
		
		
		if(giftdata && giftdata.ignore) {
			log('Ignoring '+giftname+'.');
			request(giftdata.ignore,function(msg){
				if($('#'+spocklet+'_sendback').is(':checked')) {
					if(giftdata.sendback){
						
						var url = 'xw_controller=requests&xw_action=postSend&xw_city=7&tmp=&cb=&xw_person='+User.id.substr(2)+'&mwcom=1';
						var parts = JSON.parse('[' + giftdata.sendback + ']');
						
						url = url + '&rid=' + (parseInt(Math.random()*1000000000));
						url = url + '&data=' + parts[2];
						url = url + '&to=' + parts[0]; 

						request(url,function(msg){
							try {
								var $msg=$((JSON.parse(msg)).data.reqsentmsg);
								log('Sending back gift: '+$msg.find('div:first').text());
								sentback++;
								$('#'+spocklet+'_sentback').html('<b>Gifts sent back: '+sentback+'</b>');
								try { update_game_ui_gc(JSON.parse(msg)); } catch(ignore) {}
							} catch(e) {}
							callback();
						},function(){
							log('Unknown error, retrying...');
							callback();
						});
					} else {
						log('No Thank You for this link available');
						callback();
					}
				} else {
					callback();
				}
				try { update_game_ui_gc(JSON.parse(msg)); } catch(ignore) {}
			});
		} else {
			log('Cannot ignore, continue.');
			callback();
		}
	}
	
	function push_loot(lootname,img,quantity) {
		var m;
		// if quantity given in name, use it
		if ((quantity==1) && (m=/^(\d+)\s(.*)$/.exec(lootname))) {
			lootname=m[2];
			quantity=parseInt(m[1]);
			// if quantity > 1, remove plural 's'
			if ((quantity>1) && (m=/^(.*)s$/.exec(lootname))) {
				lootname=m[1];
			}
		}
		if(!loot[lootname]) {
		loot[lootname]={ q:quantity,img:img };
		} else {
			loot[lootname].q+=quantity;
			if(!loot[lootname].img) { loot[lootname].img=img; }
		}
	}
	
	function add_loot(msg,img) {
		var m;
		try {
			log(msg);
		} catch(all) {}
		
		if(m=/You just accepted: ([^\!]+)\!/.exec(msg)) {
			push_loot(m[1],img,1);
		}
		else if (m=/Your Mystery Boost contained: a?n?\s?([^\!]+)\!/.exec(msg)) {
			var list=m[1].split(', ');
			for(var i=0;i<list.length;i++) {
				push_loot(list[i],img,1);
			}
		} else if (m=/Chill out with Mystery Boost that has: ([^\!]+)\!/.exec(msg)) {
			var list=m[1].split(', ');
			for(var i=0;i<list.length;i++) {
				push_loot(list[i],img,1);
			}	
		}
		else if(m=/Your .* contained: a?n?\s?([^\!]+)\!/.exec(msg)) {
			push_loot(m[1],img,1);
		}
		else if(m=/Here's a .* item instead a?n?\s?([^\!]+)$/.exec(msg)) {
			push_loot(m[1],img,1);
		}
		else if(m=/ You received: a?n?\s?([^\!]+)$/.exec(msg)) {
			push_loot(m[1],img,1);
		}
		else if (m=/You got: a?n?\s?([^\!]+)$/.exec(msg)) {
			push_loot(m[1],img,1);
		} 
		else if (m=/You have been added to friend's queue/.exec(msg)) {
			push_loot('City Crew',img,1);
		} 
		else if (m=/Get ready to Lord Over London/.exec(msg)) {
			push_loot('London Invite',img,1);
		}
		// loy points
		if(m=/You also earned (\d+) Loyalty Points/.exec(msg)) {
			push_loot('Loyalty Points','https://zyngapv.hs.llnwd.net/e6/mwfb/graphics/crm/CRM_LP-icon-s.png',parseInt(m[1]));
		}
		update_loot();
	}
	
	function update_loot() {
		var html='<b>Loot</b><br />',i;
		for(i in loot) {
			html+='<div><img height=19 width=19 src="'+loot[i].img+'">'+i+'<span class=good> &times; '+loot[i].q+'</span></div>';
		}
		$('#'+spocklet+'_loot').html(html);
	}
	
	function request(url, handler, errorhandler) {
		var timestamp = parseInt(new Date().getTime().toString().substring(0, 10));
		if (url.indexOf('cb=') == -1) {
			url += '&cb='+User.id+timestamp;
		}
		if (url.indexOf('tmp=') == -1) {
			url += '&tmp='+timestamp;
		}
		User.clicks++;
		var params = {
			'ajax': 1,
			'liteload': 1,
			'sf_xw_user_id': User.id,
			'sf_xw_sig': local_xw_sig,
			'xw_client_id': 8,
			'skip_req_frame': 1,
			'clicks': User.clicks
		};
		$.ajax({
			type: "POST",
			url: preurl+url,
			data: params,
			cache: false,
			success: handler,
			error: errorhandler
		});
	}	
	function update_game_ui_gc(object) {
		if (Util.isset(object.user_fields)) {
			user_fields_update(object.user_fields);
			user_info_update(object.user_fields, object.user_info);
			if (Util.isset(object.user_fields.zmc_event_count)) {
				$('#zmc_event_count').text(object.user_fields.zmc_event_count);
			}
			if (Util.isset(object.user_fields.current_city_id)) {
				active_city = object.user_fields.current_city_id;
				if (object.user_fields.current_city_id == 1 && !$('#'+spocklet+'_disableny').is(':checked')) {
					//disable new york banking
					if ($('#'+spocklet+'_bankamount').val() > 0) {
						$('#'+spocklet+'_bankamount').val('0');
						log('<span class="bad">New York bank failsafe enabled!</span>');
					}
				}
				if (object.user_fields.current_city_id != xw_city) {
					set_background(object.user_fields.current_city_id);
					xw_city = object.user_fields.current_city_id;
					// log('Travel detected! Reloading game UI.');
					// do_ajax('inner_page', 'remote/html_server.php?xw_controller=fight&xw_action=view&xw_city=', 1, 1, 0, 0);
				}
			}
		}
		if (Util.isset(object.questData)) {
			MW.QuestBar.update(object.questData);
		}
		if (Util.isset(object.fightbar)) {
			if (stats.attack_start == 0) {
				stats.attack_start = parseInt(object.fightbar.group_atk);
			}
			if (stats.defense_start == 0) {
				stats.defense_start = parseInt(object.fightbar.group_def);
			}
			stats.attack = parseInt(object.fightbar.group_atk);
			stats.defense = parseInt(object.fightbar.group_def);
		}
		if (Util.isset(object.questData) && Util.isset(object.questData.clanXp) && Util.isset(object.questData.clanXp.exp_fight)) {
			var family_ice_progress=object.questData.clanXp.exp_fight.xp + '/' + object.questData.clanXp.exp_fight.xp_max;
			$('#'+spocklet+'_ice_progress').html('['+family_ice_progress+']');
		}
	}
	/*
	 *  Function to check for parameters given to this script. i.e. run it with scriptname.js?action=kill, then do a if(get_param()['action']=='kill') { ... }
	 *
	 *  If you copy this function, change the script name. 
	 *  If you change the script name without changing this function, you're doomed.
	 *  If you copy the script without having the prototype.re, you're doomed too.
	 */	
	function get_params() {
		try {
			var foundscript;
			$('script').each(function(){
				var src=$(this).attr('src');
				if(src && (src.indexOf('giftcollector.js?')!=-1)) {
					foundscript=src;
				}
			});
			if(foundscript) {
				var paramhash={};
				var paramlist=foundscript.re('\\?(.*)$');
				var params=paramlist.split('&');
				for(var i=0;i<params.length;i++) {
					var param=params[i].split('=');
					if(param.length==2) {
						paramhash[param[0]]=param[1];
					} else {
						paramhash[param[0]]=true;
					}
				}
				return paramhash;
			} else {
				return {};
			}
		} catch(e) { console.log(e); return {}; }
	}
	// by Eike
	String.prototype.re = function(regex){
		var r=new RegExp(regex);
		var m;
		if(m=r.exec(this)) {
			return m[1];
		} else {
			return '';
		}
	}	
		
	function min(a,b){
		return a<b?a:b;
	}
	function imgurl(img,w,h,a) {
		return '<img src="http://mwfb.static.zgncdn.com/mwfb/graphics/'+img+'" width="'+w+'" height="'+h+'" title="'+a+'" alt="'+a+'" align="absmiddle">';
	}
	function commas(s) {
		while (d=/(-)?(\d+)(\d{3}.*)/.exec(s)) {
			s = (d[1]?d[1]+d[2]+','+d[3]:d[2]+','+d[3]);
		}
		return s;
	}
	function timestamp() {
		now = new Date();
		var CurH = now.getHours();
		CurH = (CurH<10?'0'+CurH:CurH);
		var CurM = now.getMinutes();
		CurM = (CurM<10?'0'+CurM:CurM);
		var CurS = now.getSeconds();
		CurS = (CurS<10?'0'+CurS:CurS);
		return '<span class="more_in">['+CurH+':'+CurM+']</span> ';
	}	

	function log(message){
		message='<span class="more_in">'+timestamp()+'</span> '+message;
		var limit = parseInt($('#'+spocklet+'_loglines').val());
		logs.unshift(message);
		if (limit > 0) {
			if (logs.length>limit) {
				message=logs.pop();
			}
		}
		$('#'+spocklet+'_log').html(logs.join('<br />'));
	}	
	
	$('#'+spocklet+'_close').click(function() {
		run = false;
		$('#'+spocklet+'_main').remove();
	});
	

	//add analytics
	function loadContent(file){
		var head = document.getElementsByTagName('head').item(0);
		var scriptTag = document.getElementById('loadScript');
		if(scriptTag) head.removeChild(scriptTag);
			script = document.createElement('script');
			script.src = file;
			script.type = 'text/javascript';
			script.id = 'loadScript';
			head.appendChild(script);
	}
	loadContent('https://www.google-analytics.com/ga.js');
	try {
		var pageTracker = _gat._getTracker("UA-8435065-3");
		pageTracker._trackPageview();
		pageTracker._trackPageview("/script/GiftCollect");
	}
	catch(err) {}
	//end analytics
})()

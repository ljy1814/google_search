// ==UserScript==
// @namespace   https://greasyfork.org/users/39670

// @name  百度Baidu网盘HTTP转HTTPS
// @description 将百度网盘的链接(http://pan.baidu.com/)转换为HTTPS协议(https://pan.bidu.com)，可以直接下载大文件。

// @author      Netplaier
// @version     1.01
// @license     LGPLv3

// @include     http://pan.baidu.com/*
// @include     http://yun.baidu.com/*

// @grant       none

// @icon        https://pan.baidu.com/res/static/images/favicon.ico

// ==/UserScript==


(function(){
  var debug = 0;
  var new_location = location.href.replace(/http\:/, 'https:');
  if ( debug > 0 ) {
    alert(  "Hash:     "+location.hash+
          "\nHost:     "+location.host+
          "\nHostname: "+location.hostname+
          "\nHREF:     "+location.href+
          "\nPathname: "+location.pathname+
          "\nPort:     "+location.port+
          "\nProtocol: "+location.protocol+
          "\n"+
          "\nNew Location: "+new_location);
  };
  location.href = new_location;
})();


// ==UserScript==
// @name        百度网盘助手•改
// @author      有一份田
// @description 显示百度网盘文件的直接链接,突破大文件需要使用电脑管家的限制
// @namespace   https://greasyfork.org/zh-CN/scripts/986-百度网盘助手
// @icon        http://img.duoluohua.com/appimg/script_dupanlink_icon_48.png
// @license     GPL version 3
// @encoding    utf-8
// @date        26/08/2013
// @modified    07/18/2016
// @include     https://pan.baidu.com/*
// @include     http://pan.baidu.com/*
// @include     http://yun.baidu.com/*
// @exclude     http://yun.baidu.com
// @exclude     http://yun.baidu.com/#*
// @exclude     http://pan.baidu.com/share/manage*
// @exclude     http://pan.baidu.com/disk/recyclebin*
// @exclude     http://yun.baidu.com/pcloud/album/info*
// @require     http://code.jquery.com/jquery-2.1.1.min.js
// @grant       unsafeWindow
// @grant       GM_setClipboard
// @run-at      document-end
// @version     3.1.0
// ==/UserScript==


/*
 * === 说明 ===
 *@作者:有一份田
 *@官网:http://www.duoluohua.com/download/
 *@Email:youyifentian@gmail.com
 *@Git:http://git.oschina.net/youyifentian
 *@转载重用请保留此信息
 *
 *
 * */





var VERSION = '3.0.2';
var APPNAME = '\u767e\u5ea6\u7f51\u76d8\u52a9\u624b';
var t = new Date().getTime();


$ = $ || unsafeWindow.$;
var disk = unsafeWindow.disk;
var FileUtils = unsafeWindow.FileUtils;
var Page = unsafeWindow.Page;
var Utilities = unsafeWindow.Utilities;
var yunData = unsafeWindow.yunData;
var require= unsafeWindow.require;


(function (){
    var isOther = location.href.indexOf('://pan.baidu.com/disk')==-1,
    downProxy = null,shareData = null,
    Canvas,Pancel,RestAPI,Toast={},errorMsg,
    iframe = '',httpHwnd = null,index = 0,
    msg = [
        '\u54b1\u80fd\u4e0d\u4e8c\u4e48,\u4e00\u4e2a\u6587\u4ef6\u90fd\u4e0d\u9009\u4f60\u8ba9\u6211\u548b\u4e2a\u529e...', //0
        '\u5c3c\u739b\u4e00\u4e2a\u6587\u4ef6\u90fd\u4e0d\u9009\u4f60\u4e0b\u4e2a\u6bdb\u7ebf\u554a...', //1
        '\u4f60TM\u77e5\u9053\u4f60\u9009\u4e86<b>90</b>\u591a\u4e2a\u6587\u4ef6\u5417?\u60f3\u7d2f\u6b7b\u6211\u554a...', //2
        '<b>\u8bf7\u6c42\u5df2\u53d1\u9001\uff0c\u6570\u636e\u4e0b\u884c\u4e2d...</b>', //3
        '<b>\u8be5\u9875\u9762</b>\u4e0d\u652f\u6301\u6587\u4ef6\u5939\u548c\u591a\u6587\u4ef6\u7684<font color="red"><b>\u94fe\u63a5\u590d\u5236\u548c\u67e5\u770b</b></font>\uff01', //4
        '<font color="red">\u8bf7\u6c42\u8d85\u65f6\u4e86...</font>', //5
        '<font color="red">\u8bf7\u6c42\u51fa\u9519\u4e86...</font>', //6
        '<font color="red">\u8fd4\u56de\u6570\u636e\u65e0\u6cd5\u76f4\u89c6...</font>', //7
        '\u8bf7\u8f93\u5165\u9a8c\u8bc1\u7801', //8
        '\u9a8c\u8bc1\u7801\u8f93\u5165\u9519\u8bef,\u8bf7\u91cd\u65b0\u8f93\u5165', //9
        '<b>\u94fe\u63a5\u5df2\u590d\u5236\u5230\u526a\u5207\u677f\uff01</b>', //10
        '\u672a\u77e5\u9519\u8bef\uff0cerrno:',//11
        '<font color="red"><b>\u5c3c\u739b\u7adf\u7136\u8dea\u4e86\u4e86\uff0c\u4e0d\u8981\u544a\u8bc9\u6211\u4f60\u7684\u6c34\u8868\u5728\u91cc\u9762...</b></font>',//12
        ''
        ],
    btnClassArr=[
        {css:'icon-download',tag:'a',id:''},
        {css:'icon-btn-download',tag:'li',id:''},
        {css:'icon-btn-download',tag:'',id:''},
        {css:'download-btn',tag:'',id:''},
        {css:'',tag:'',id:'downFileButton'}
    ];
    try{
        downProxy = isOther ? disk.util.DownloadProxy || null : null;
        shareData = isOther ? disk.util.ViewShareUtils || null : null;
    }catch(e){}
    if(!isOther || (isOther && !FileUtils)){
        RestAPI=require("common:widget/restApi/restApi.js");
        Canvas=require("common:widget/canvasPanel/canvasPanel.js");
        Pancel=require("common:widget/panel/panel.js");
        Toast = require("common:widget/toast/toast.js");
        errorMsg = require("common:widget/errorMsg/errorMsg.js");
    }
    var helperMenuBtns=(function(){
        var menuTitleArr=['\u76f4\u63a5\u4e0b\u8f7d','\u590d\u5236\u94fe\u63a5','\u67e5\u770b\u94fe\u63a5'],panBtnsArr=[],html='';
        for(var i=0;i<btnClassArr.length;i++){
            var item=btnClassArr[i];
            var tmpItem=item.id!='' ? $('#'+item.id) : $('.'+item.css);
            var tmpArr=item.tag!='' ? tmpItem.parent(item.tag) : tmpItem;
            panBtnsArr=$.merge(panBtnsArr,tmpArr.toArray());
        }
        if(!panBtnsArr.length){return panBtnsArr;}
        html+='<div id="panHelperMenu" style="display:none;position:fixed;z-index:999999;">';
        html+='<ul class="pull-down-menu" style="display:block;margin:0px;padding:0px;left:0px;top:0px;list-style:none;">';
        for(var i=0;i<menuTitleArr.length;i++){
            html+='<li><a href="javascript:;" class="panHelperMenuBtn" type="'+i+'"><b>'+menuTitleArr[i]+'</b></a></li>';
        }
        html+='<li style="display:none;"><a href="' + getApiUrl('getnewversion', 1) + '" target="_blank">';
        html+='<img id="updateimg" title="\u6709\u4e00\u4efd\u7530" style="border:none;"/></a></li></ul></div>';
        $('<div>').html(html).appendTo(document.body);
        for (var i = 0; i < panBtnsArr.length; i++) {
            var item = panBtnsArr[i];
            createHelperBtn(item);
        }
        function createHelperBtn(btn) {
            var newnode=btn.cloneNode(true),html=newnode.innerHTML;
            $(newnode).attr('id','').attr('href','javascript:void(0)').attr('data-key','downLoadhelper').attr('node-type','btnHelper').attr('onclick','').css({width:63}).html(html.replace(/[\u4E00-\u9FA5]{2,4}(\(.*\)|\uff08.*\uff09)?/,'\u7f51\u76d8\u52a9\u624b')).unbind();
            var o=$('<div class="PanHelperBtn" style="display:inline-block;">').append(newnode)[0];
            btn.parentNode.insertBefore(o, btn.nextSibling);
            return o;
        }
        var helperBtn = $('.PanHelperBtn'),helperMenu = $('#panHelperMenu'),
        menuFun = function() {
            helperDownload($(this).attr('type') || 0);
            helperMenu.hide();
        };
        helperBtn.click(menuFun).mouseenter(function() {
            $(this).addClass('b-img-over');
            var o=$(this).children('a'),offset=o.offset(),w=o.outerWidth()-parseInt(o.css('paddingRight'));
            helperMenu.children('ul').css('width', w-2);
            helperMenu.css('top', offset.top + o.height() + parseInt(o.css('paddingTop')) - $(document).scrollTop());
            helperMenu.css('left', offset.left).show();
        }).mouseleave(function() {
            $(this).removeClass('b-img-over');
            helperMenu.hide();
        });
        $(document).scroll(function() {
            helperMenu.hide();
        });
        helperMenu.mouseenter(function() {
            $(this).show();
        }).mouseleave(function() {
            $(this).hide();
        });
        helperMenu.find('a').css('text-align', 'center');
        return helperMenu.find('a.panHelperMenuBtn').click(menuFun).toArray();
    })();
    if(!helperMenuBtns.length){return;}
    checkUpdate();
    function helperDownload(type){
        iframe=createDownloadIframe();
        iframe.src = 'javascript:;';
        var items = getListViewCheckedItems(),len = items.length;
        if(!len) {
            index = 1 == index ? 0 : 1;
            return myToast(msg[index]);
        }else if (len > 90) {
            return myToast(msg[2]);
        }
        if(1 == len) {
            var url = items[0].dlink;
            if(isUrl(url)) {
                if(2 == type) {
                    showHelperDialog(type, items, {"errno": 0,"dlink": url});
                }else if(1 == type){
                    copyText(url);
                }else{
                    myToast(msg[3],1);
                    iframe.src = url;
                }
            }else{
                getDownloadInfo(type, items);
            }
        }else {
            getDownloadInfo(type, items);
        }
        downloadCounter(items);
    }
    function getDownloadInfo(type, items, vcode) {
        if(!vcode) {
            showHelperDialog(helperMenuBtns.length+1, items);
            vcode = {};
        }
        var url = '',data = {},fidlist = '',fids = [];
        for (var i = 0; i < items.length; i++) {
            fids.push(items[i]['fs_id']);
        }
        fidlist = '[' + fids.join(',') + ']';
        if(isOther){
            if(FileUtils){
                url = disk.api.RestAPI.SHARE_GET_DLINK + '&uk=' + FileUtils.share_uk + '&shareid=' + FileUtils.share_id + '&timestamp=' + FileUtils.share_timestamp + '&sign=' + FileUtils.share_sign + '&fid_list=' + fidlist;
                data = {
                    shareid:FileUtils.share_id,
                    uk:FileUtils.share_uk,
                    fid_list:fidlist
                };
            }else{
                var context=yunData.getContext();
                if(typeof vcode =='object'){
                    url = '/api/sharedownload?' + 'uk=' + yunData.SHARE_UK + '&shareid=' + yunData.SHARE_ID + '&timestamp=' + yunData.TIMESTAMP + '&sign=' + yunData.SIGN + '&fid_list=' + fidlist;
                    data = 'encrypt=0&product=share&primaryid=' + yunData.SHARE_ID + '&shareid=' + yunData.SHARE_ID + '&uk=' + yunData.SHARE_UK + '&fid_list=' + fidlist+ '&extra=' + '{"sekey":"' + context.sekey + '"}';
                    data = {
                        encrypt:0,
                        extra:'{"sekey":"' + context.sekey + '"}',
                        product:'share',
                        primaryid:yunData.SHARE_ID,
                        shareid:yunData.SHARE_ID,
                        uk:yunData.SHARE_UK,
                        fid_list:fidlist
                    };
                }else{
                    url = RestAPI.GET_CAPTCHA + '?prod=share';
                }
            }
            if(typeof vcode =='object'){
                data = $.extend(data,vcode);
            }
            data.type=(items.length >1 || items[0]['isdir']) ? "batch" : "dlink";
        }else{
            url = RestAPI.DOWN_GET_DLINK;
            if ("function" != typeof yunData.sign2) try {
                yunData.sign2 = new Function("return " + yunData.sign2)();
            } catch (o) {}
            data={
                sign: base64Encode(yunData.sign2(yunData.sign3, yunData.sign1)),
                timestamp: yunData.timestamp,
                bdstoken: yunData.MYBDSTOKEN,
                fidlist: fidlist,
                type: (items.length >1 || items[0]['isdir']) ? "batch" : "dlink"
            };
        }
        httpHwnd = $.post(url, data,
            function(o) {
                var dlink = typeof o.dlink =='object' ? o.dlink[0]['dlink'] : o.dlink;
                if(-20 === o.errno){
                    getDownloadInfo(type, items, JSON.stringify(vcode) =='{}' ? 'getvcode' : 'showvcode');
                }else if (0 === o.errno) {
                    if(o.list || dlink){
                        if(!dlink){
                            var list = o.list,opt=list[0];
                            dlink=opt.dlink;
                        }
                        dlink = dlink + '&zipname=' + encodeURIComponent(getDownloadName(items));
                        o.dlink = dlink;
                        if (shareData) {
                            var obj = JSON.parse(shareData.viewShareData);
                            obj.dlink = dlink;
                            shareData.viewShareData = JSON.stringify(obj);
                        }
                        if (1 == items.length) {
                            items[0]['dlink'] = dlink;
                            if(items[0]['item']){
                                $(items[0]['item']).attr('dlink',dlink);
                            }
                            try{
                                if(yunData.SHAREPAGETYPE == "single_file_page"){
                                    yunData.FILEINFO = items;
                                }
                            }catch(e){}
                        }
                    }else{
                        if(o.vcode_img && o.vcode_str){
                            o.errno = -20;
                            if(vcode == 'getvcode'){
                                //return getDownloadInfo(type, items, getVCode('test',o.vcode_str));
                            }
                        }
                    }
                    showHelperDialog(type, items, o, vcode);
                }else{
                    showHelperDialog(type, items, o, vcode);
                }
            });
    }
    function showHelperDialog(type, items, opt, vcode) {
        var canvas =document.canvas ? document.canvas : Canvas ? new Canvas() : new disk.ui.Canvas(),
        _ = document.helperdialog || createHelperDialog(),isVisible = _.isVisible(),status=0;
        document.canvas = canvas;
        _.canvas = canvas;
        _.type = type;
        _.items = items;
        if (type < helperMenuBtns.length) {
            if (0 === opt.errno) {
                status=1;
                if(type < 2) {
                    _.canvas.setVisible(false);
                    _.setVisible(false);
                    if(0 == type){
                        iframe.src = opt.dlink;
                        myToast(msg[3],1);
                    } else {
                        copyText(opt.dlink);
                    }
                    return;
                }
                _.sharefilename.innerHTML = getDownloadName(items);
                _.sharedlink.value = opt.dlink;
                _.dlink = opt.dlink;
                //_.downloadbtn.href= opt.dlink;
                _.focusobj = _.sharedlink;
            } else if(-19 ==opt.errno) {
                status=2;
                _.vcodeimg.src = opt.img;
                _.vcodeimgsrc = opt.img;
                _.vcodevalue = opt.vcode;
                _.vcodetip.innerHTML = vcode ? msg[9] : '';
                _.vcodeinput.value = '';
                _.focusobj = _.vcodeinput;
            }else if(-20 ==opt.errno) {
                status=2;
                _.vcodeimg.src = opt.vcode_img;
                _.vcodeimgsrc = opt.vcode_img;
                _.vcodevalue = opt.vcode_str;
                _.vcodetip.innerHTML = vcode && vcode!='getvcode' ? msg[9] : '';
                _.vcodeinput.value = '';
                _.focusobj = _.vcodeinput;
            } else {
                _.canvas.setVisible(false);
                _.setVisible(false);
                return myToast(errorMsg ? errorMsg.ErrorMessage[opt.errno] : disk.util.shareErrorMessage[opt.errno] || (msg[11] + opt.errno));
            }
        }
        _.loading.style.display = 0==status ? '' : 'none';
        _.showdlink.style.display = 1==status ? '' : 'none';
        _.showvcode.style.display = 2==status ? '' : 'none';
        _.copytext.style.display = 1==status ? '' : 'none';
        if (!isVisible) {
            _.canvas.setVisible(true);
            _.setVisible(true);
        }
        _.setGravity(Pancel ? Pancel.CENTER : disk.ui.Panel.CENTER);
        _.focusobj.focus();
    }
    function createHelperDialog() {
        var html = '<div class="dlg-hd b-rlv"title="\u6709\u4e00\u4efd\u7530"><span title="\u5173\u95ed"id="helperdialogclose"class="dlg-cnr dlg-cnr-r"></span><h3><a href="'+getApiUrl('getnewversion',1)+'"target="_blank"style="color:#000;">'+APPNAME+'&nbsp;' + VERSION + '</a><a href="javascript:;"title="\u70b9\u6b64\u590d\u5236"id="copytext"style="float:right;margin-right:240px;display:none;">\u70b9\u6b64\u590d\u5236</a></h3></div><div class="download-mgr-dialog-msg center"id="helperloading"><b>\u6570\u636e\u8d76\u6765\u4e2d...</b></div><div id="showvcode"style="text-align:center;display:none;"><div class="dlg-bd download-verify"style="text-align:center;margin-top:25px;"><div class="verify-body">\u8bf7\u8f93\u5165\u9a8c\u8bc1\u7801\uff1a<input type="text"maxlength="4"class="input-code vcode"><img width="100"height="30"src=""alt="\u9a8c\u8bc1\u7801\u83b7\u53d6\u4e2d"class="img-code"><a class="underline"href="javascript:;">\u6362\u4e00\u5f20</a></div><div class="verify-error"style="text-align:left;margin-left:84px;"></div></div><br><div><div class="alert-dialog-commands clearfix"><a href="javascript:;"class="sbtn okay postvcode"><b>\u786e\u5b9a</b></a><a href="javascript:;"class="dbtn cancel"><b>\u5173\u95ed</b></a></div></div></div><div id="showdlink"style="text-align:center;display:none;"><div class="dlg-bd download-verify"><div style="padding:5px 0px;"><b><span id="sharefilename"></span></b></div><input type="text"name="sharedlink"id="sharedlink"class="input-code"maxlength="1024"value=""style="width:500px;border:1px solid #7FADDC;padding:3px;height:24px;"></div><br><div><div class="alert-dialog-commands clearfix"><a href="javascript:;"class="sbtn okay postdownload"><b>\u76f4\u63a5\u4e0b\u8f7d</b></a><a href="javascript:;"class="dbtn cancel"><b>\u5173\u95ed</b></a></div></div></div>',
        o=$('<div class="b-panel download-mgr-dialog helperdialog" style="width:550px;">').html(html).appendTo(document.body);
        o[0].pane = o[0];
        var _ = Pancel ? new Pancel(o[0]) : new disk.ui.Panel(o[0]),vcodeimg = o.find('img')[0],vcodeinput = o.find('.vcode')[0],
        sharedlink = o.find('#sharedlink')[0],vcodetip = o.find('.verify-error')[0],
        copytext= o.find('#copytext')[0],postdownloadBtn=o.find('.postdownload')[0],
        dialogClose = function() {
            vcodeinput.value = '';
            vcodetip.innerHTML = '';
            vcodeimg.src = '';
            _.canvas.setVisible(false);
            _.setVisible(false);
            if (httpHwnd) {httpHwnd.abort();}
        },
        postvcode = function() {
            if (httpHwnd) {httpHwnd.abort();}
            var v = vcodeinput.value,len = v.length,max = msg.length - 1,i = max,
            vcode = getVCode(v,_.vcodevalue);
            i = 0 == len ? 8 : (len < 4 ? 9 : i);
            vcodetip.innerHTML = msg[i];
            if (i != max) {return vcodeinput.focus();}
            getDownloadInfo(_.type, _.items, vcode);
        },
        postdownload = function(e) {
            //if(!e){iframe.src = _.dlink;}
            iframe.src = _.dlink;
            dialogClose();
            myToast(msg[3],1);
        };
        _._mUI.pane = o[0];
        _.loading = o.find('#helperloading')[0];
        _.showvcode = o.find('#showvcode')[0];
        _.showdlink = o.find('#showdlink')[0];
        _.copytext= copytext;
        _.downloadbtn=postdownloadBtn;
        _.vcodeinput = vcodeinput;
        _.sharedlink = sharedlink;
        _.sharefilename = o.find('#sharefilename')[0];
        _.vcodeimg = vcodeimg;
        _.vcodetip = vcodetip;
        _.vcodeimgsrc = '';
        _.vcodevalue = '';
        _.focusobj = sharedlink;
        $(copytext).click(function(){
            copyText(_.dlink);
            this.blur();
        });
        $(vcodeimg).siblings('a').click(function() {
            vcodeimg.src = _.vcodeimgsrc + '&' + new Date().getTime();
            vcodeinput.focus();
        });
        vcodeinput.onkeydown = function(e) {
            if (13 == e.keyCode) {postvcode();}
        };
        o.find('.postvcode').click(postvcode);
        $(postdownloadBtn).click(postdownload);
        $('#sharedlink').focusin(function() {
            this.style.boxShadow = '0 0 3px #7FADDC';
            this.select();
        }).focusout(function() {
            this.style.boxShadow = '';
        }).mouseover(function() {
            this.select();
            this.focus();
        }).keydown(function(e) {
            if (13 == e.keyCode) {postdownload();}
        });
        $(window).bind("resize",function() {
            _.setGravity(Pancel ? Pancel.CENTER : disk.ui.Panel.CENTER);
        });
        o.find('#helperdialogclose').click(dialogClose);
        o.find('.dbtn').click(dialogClose);
        _.setVisible(false);
        document.helperdialog = _;
        return _;
    }
    function getVCode(v,k){
        return FileUtils ? {input:v,vcode:_.k} : {vcode_input:v,vcode_str:k};
    }
    function myToast(msg, type) {
        try{
            unsafeWindow.myToastInjection(msg,type,isOther);
            return;
        }catch(e){}
        try {
            var Toast = {}, obtain,Pancel = null;
            if (isOther && disk.ui) {
                obtain = disk.ui.Toast;
                Toast.obtain = {};
                Toast.obtain.useToast = Utilities.useToast;
            } else {
                Toast = require("common:widget/toast/toast.js");
                Pancel=require("common:widget/panel/panel.js");
                obtain = Toast.obtain;
            }
            var o = Toast.obtain.useToast({
                toastMode:type ? obtain.MODE_SUCCESS :obtain.MODE_FAILURE,
                msg:msg,
                sticky:false,
                position:Pancel ? Pancel.TOP : (disk.ui ? disk.ui.Panel.TOP : undefined)
            });
            try {
                $(o._mUI.pane).css({
                    "z-index":999999
                });
            } catch (e) {}
        } catch (err) {
            if (!type) {
                alert(msg);
            }
        }
    }
    function copyText(text){
        GM_setClipboard(text);
        myToast(msg[10],1);
    }
    function createDownloadIframe(){
        if(iframe){return iframe;}
        var o = $('#helperdownloadiframe');
        iframe=o.length ? o[0] : '';
        if(!iframe) {
            iframe = $('<div style="display:none;">').html('<iframe src="" id="helperdownloadiframe" name="helperdownloadiframe"></iframe>').appendTo(document.body).find('#helperdownloadiframe')[0];
        }
        $(iframe).load(function(){
            if(this.src=='javascript:;'){return;}
            myToast(msg[12],0);
        });
        return iframe;
    }
    function getListViewCheckedItems(){
        var items=[];
        if(shareData){
            items.push(JSON.parse(shareData.viewShareData));
        }else if(isOther) {
            if(FileUtils){
                items = FileUtils.getListViewCheckedItems();
            }else if(yunData){
                if(yunData.SHAREPAGETYPE == "multi_file"){
                    items=getCheckItems();
                }else if(yunData.SHAREPAGETYPE == "single_file_page"){
                    items=yunData.FILEINFO;
                }
            }
        }else{
            items=getCheckItems();
        }
        return items;
    }
    function getCheckItems(){
        var items=[],boxCss=$('.list-selected').length ? 'module-list-view' : 'module-grid-view';
        $('div.' + boxCss).find('.item-active').each(function(i,o){
            items.push(getListViewCheckedItemInfo(o));
        });
        return items;
    }
    function getListViewCheckedItemInfo(obj){
        var o=$(obj),fs_id=o.attr('data-id'),category=o.attr('data-category'),
            isdir=o.attr('data-extname')=='dir' ? 1 : 0,
            server_filename=o.find('[node-type="name"]').attr('title'),
            dlink=o.attr('dlink') || '';
        return {'fs_id':fs_id,'category':category,'isdir':isdir,'server_filename':server_filename,'dlink':dlink,'item':obj};
    }
    function getDownloadName(items) {
        var packName=items[0]['server_filename'];
        if (items.length > 1 || 1 == items[0]['isdir']) {
            try{
                downProxy.prototype.setPackName(FileUtils.parseDirFromPath(items[0]['path']), !items[0]['isdir']);
                packName= downProxy.prototype._mPackName;
            }catch(e){
                packName='\u3010\u6279\u91cf\u4e0b\u8f7d\u3011'+packName+'\u7b49.zip';
            }
        }
        return packName;
    }
    function downloadCounter(C) { //C:items,B:isOneFile
        if (!isOther) {return;}
        var F = FileUtils ? FileUtils.share_uk || disk.util.ViewShareUtils.uk : yunData.SHARE_UK,
        D = FileUtils ? FileUtils.share_id : yunData.SHARE_ID,
        A = [],B = (1 == C.length && 0 == C[0].isdir),
        G = shareData ? disk.util.ViewShareUtils.albumId: '';
        for (var _ in C) {
            if (C.hasOwnProperty(_)) {
                var E = {
                    fid: C[_].fs_id,
                    category: C[_].category
                };
                A.push(E);
            }
        }
        G && B && $.post(disk.api.RestAPI.PCLOUD_ALBUM_DOWNLOAD_COUNTER, {
            uk: F,
            album_id: G,
            fs_id: C[_].fs_id
        });
        !G && $.post(FileUtils ? disk.api.RestAPI.MIS_COUNTER : RestAPI.MIS_COUNTER, {
            uk: F,
            filelist: JSON.stringify(A),
            sid: D,
            ctime: FileUtils ? FileUtils.share_ctime : yunData.SHARE_TIME,
            "public": FileUtils ? FileUtils.share_public_type : yunData.SHAREPAGETYPE,
            t: (new Date).getTime(),
            _: Math.random()
        });
        !G && B && $.get(FileUtils ? disk.api.RestAPI.SHARE_COUNTER : RestAPI.SHARE_COUNTER, {
            type: 1,
            shareid: D,
            uk: F,
            sign: FileUtils ? FileUtils.share_sign : yunData.SIGN,
            timestamp: FileUtils ? FileUtils.share_timestamp : yunData.TIMESTAMP,            
            t: new Date().getTime(),
            _: Math.random()
        });
    }
})();
loadJs('function myToastInjection(msg,type,isOther){try{var Toast={},obtain,Pancel=null;if(isOther&&disk.ui){obtain=disk.ui.Toast;Toast.obtain={};Toast.obtain.useToast=Utilities.useToast}else{Toast=require("common:widget/toast/toast.js");Pancel=require("common:widget/panel/panel.js");obtain=Toast.obtain}var o=Toast.obtain.useToast({toastMode:type?obtain.MODE_SUCCESS:obtain.MODE_FAILURE,msg:msg,sticky:false,position:Pancel?Pancel.TOP:(disk.ui?disk.ui.Panel.TOP:undefined)});try{$(o._mUI.pane).css({"z-index":999999})}catch(e){}}catch(err){if(!type){alert(msg)}}}');
function base64Encode(a){var b,c,d,e,f,g,h="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";for(d=a.length,c=0,b="";d>c;){if(e=255&a.charCodeAt(c++),c==d){b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4),b+="==";break}if(f=a.charCodeAt(c++),c==d){b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4|(240&f)>>4),b+=h.charAt((15&f)<<2),b+="=";break}g=a.charCodeAt(c++),b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4|(240&f)>>4),b+=h.charAt((15&f)<<2|(192&g)>>6),b+=h.charAt(63&g)}return b;}
function isUrl(url) {
    return /^(http|https):\/\/([\w-]+(:[\w-]+)?@)?[\w-]+(\.[\w-]+)+(:[\d]+)?([#\/\?][^\s<>;"\']*)?$/.test(url);
}
function checkUpdate() {
    var js = 'var upinfo=document.getElementById("updateimg");';
    js += 'upinfo.src="' + getApiUrl('checkupdate', 1) + '";';
    js += 'upinfo.onload=function(){';
    js += 'upinfo.parentNode.parentNode.style.display="";';
    js += '}';
    loadJs(js);
}
function getApiUrl(action, type) {
    return 'http://app.duoluohua.com/update?action=' + action + '&system=script&appname=dupanlink&apppot=scriptjs&frompot=dupan&type=' + type + '&version=' + VERSION + '&t=' + t;
}
function loadJs(js) {
    var oHead = document.getElementsByTagName('HEAD')[0],
    oScript = document.createElement('script');
    oScript.type = 'text/javascript';
    oScript.text = js;
    oHead.appendChild(oScript);
}
function googleAnalytics() {
    var js = "var _gaq = _gaq || [];";
    js += "_gaq.push(['_setAccount', 'UA-43859764-1']);";
    js += "_gaq.push(['_trackPageview']);";
    js += "function googleAnalytics(){";
    js += "	var ga = document.createElement('script');ga.type = 'text/javascript';";
    js += "	ga.async = true;ga.src = 'https://ssl.google-analytics.com/ga.js';";
    js += "	var s = document.getElementsByTagName('script')[0];";
    js += "	s.parentNode.insertBefore(ga, s)";
    js += "}";
    js += "googleAnalytics();";
    js += "_gaq.push(['_trackEvent','dupanlink_script',String('" + VERSION + "')]);";
    loadJs(js);
}
googleAnalytics();

// ==UserScript==
// @name         百度网盘直接下载助手
// @namespace    undefined
// @version      0.9.24
// @description  直接下载百度网盘和百度网盘分享的文件,避免下载文件时调用百度网盘客户端,获取网盘文件的直接下载地址
// @author       ivesjay
// @match        *://pan.baidu.com/disk/home*
// @match        *://yun.baidu.com/disk/home*
// @match        *://pan.baidu.com/s/*
// @match        *://yun.baidu.com/s/*
// @match        *://pan.baidu.com/share/link*
// @match        *://yun.baidu.com/share/link*
// @require      https://code.jquery.com/jquery-latest.js
// @run-at       document-start
// @grant        unsafeWindow
// @grant        GM_setClipboard
// ==/UserScript==

(function() {
    'use strict';

    var $ = $ || window.$;
    var log_count = 1;
    var wordMapHttp = {
        'list-grid-switch':'yvgb9XJ',
        'list-switched-on':'ksbXZm',
        'grid-switched-on':'tch6W25',
        'list-switch':'lrbo9a',
        'grid-switch':'xh6poL',
        'checkbox':'EOGexf',
        'col-item':'Qxyfvg',
        'check':'fydGNC',
        'checked':'EzubGg',
        'list-view':'vdAfKMb',
        'item-active':'ngb9O6',
        'grid-view':'JKvHJMb',
        'bar-search':'OFaPaO',
        'default-dom':'xpX2PV',
        'bar':'qxnX2G5',
        'list-tools':'QDDOQB'
    };
    var wordMapHttps = {
        'list-grid-switch':'qobmXB1q',
        'list-switched-on':'ewXm1e',
        'grid-switched-on':'kxhkX2Em',
        'list-switch':'rvpXm63',
        'grid-switch':'mxgdJgwv',
        'checkbox':'EOGexf',
        'col-item':'Qxyfvg',
        'check':'fydGNC',
        'checked':'EzubGg',
        'list-view':'vdAfKMb',
        'item-active':'pcamXBRX',
        'grid-view':'JKvHJMb',
        'bar-search':'OFaPaO',
        'default-dom':'nyztJqWE',
        'bar':'mkseJqKQ',
        'list-tools':'QDDOQB'
    };
    var wordMap = location.protocol == 'http:' ? wordMapHttp : wordMapHttps;
    
    //console.log(wordMap);

    function slog(c1,c2,c3){
        c1 = c1?c1:'';
        c2 = c2?c2:'';
        c3 = c3?c3:'';
        console.log('#'+ log_count++ +'-BaiDuNetdiskHelper-log:',c1,c2,c3);
    }

    $(function(){
        switch(detectPage()){
            case 'disk':
                var panHelper = new PanHelper();
                panHelper.init();
                return;
            case 'share':
            case 's':
                var panShareHelper = new PanShareHelper();
                panShareHelper.init();
                return;
            default:
                return;
        }
    });

    //网盘页面的下载助手
    function PanHelper(){
        var yunData,sign,timestamp,bdstoken,logid,fid_list;
        var fileList=[],selectFileList=[],batchLinkList=[],batchLinkListAll=[],linkList=[],
            list_grid_status='list';
        var observer,currentPage,currentPath,currentCategory,dialog,searchKey;
        var panAPIUrl = location.protocol + "//" + location.host + "/api/";
        var restAPIUrl = location.protocol + "//pcs.baidu.com/rest/2.0/pcs/";
        var clientAPIUrl = location.protocol + "//d.pcs.baidu.com/rest/2.0/pcs/";

        this.init = function(){
            yunData = unsafeWindow.yunData;
            slog('yunData:',yunData);
            if(yunData === undefined){
                slog('页面未正常加载，或者百度已经更新！');
                return;
            }
            initParams();
            registerEventListener();
            createObserver();
            addButton();
            createIframe();
            dialog = new Dialog({addCopy:true});

            slog('网盘直接下载助手加载成功！');
        };

        function initParams(){
            sign = getSign();
            timestamp = getTimestamp();
            bdstoken = getBDStoken();
            logid = getLogID();
            currentPage = getCurrentPage();
            slog('Current display mode:',currentPage);

            if(currentPage == 'list')
                currentPath = getPath();

            if(currentPage == 'category')
                currentCategory = getCategory();

            if(currentPage == 'search')
                searchKey = getSearchKey();

            refreshListGridStatus();
            refreshFileList();
            refreshSelectList();
        }

        function refreshFileList(){
            if (currentPage == 'list') {
                fileList = getFileList();
            } else if (currentPage == 'category'){
                fileList = getCategoryFileList();
            } else if (currentPage == 'search') {
                fileList = getSearchFileList();
            }
        }

        function refreshSelectList(){
            selectFileList = [];
        }

        function refreshListGridStatus(){
            list_grid_status = getListGridStatus();
        }

        //获取当前的视图模式
        function getListGridStatus(){
            //return $('div.list-grid-switch').hasClass('list-switched-on')?'list':($('div.list-grid-switch').hasClass('grid-switched-on')?'grid':'list');
            //return $('div.itiWzPY').hasClass('kudtWY46')?'list':($('div.itiWzPY').hasClass('nytAL9w')?'grid':'list');
            return $('div.'+wordMap['list-grid-switch']).hasClass(wordMap['list-switched-on'])?'list':($('div.'+wordMap['list-grid-switch']).hasClass(wordMap['grid-switched-on'])?'grid':'list');
        }

        function registerEventListener(){
            registerHashChange();
            registerListGridStatus();
            registerCheckbox();
            registerAllCheckbox();
            registerFileSelect();
        }

        //监视地址栏#标签的变化
        function registerHashChange(){
            window.addEventListener('hashchange',function(e){
                refreshListGridStatus();
                if(getCurrentPage() == 'list') {
                    if(currentPage == getCurrentPage()){
                        if(currentPath == getPath()){
                            return;
                        } else {
                            currentPath = getPath();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        currentPath = getPath();
                        refreshFileList();
                        refreshSelectList();
                    }
                } else if (getCurrentPage() == 'category') {
                    if(currentPage == getCurrentPage()){
                        if(currentCategory == getCategory()){
                            return;
                        } else {
                            currentPage = getCurrentPage();
                            currentCategory = getCategory();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        currentCategory = getCategory();
                        refreshFileList();
                        refreshSelectList();
                    }
                } else if(getCurrentPage() == 'search') {
                    if(currentPage == getCurrentPage()){
                        if(searchKey == getSearchKey()){
                            return;
                        } else {
                            currentPage = getCurrentPage();
                            searchKey = getSearchKey();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        searchKey = getSearchKey();
                        refreshFileList();
                        refreshSelectList();
                    }
                }
            });
        }

        //监视视图变化
        function registerListGridStatus(){
            //var $a_list = $('a[node-type=list-switch]');
            //var $a_list = $('a[node-type=eepWzkk]');
            var $a_list = $('a[node-type='+wordMap['list-switch']+']');
            $a_list.click(function(){
                list_grid_status = 'list';
            });

            //var $a_grid = $('a[node-type=grid-switch]');
            //var $a_grid = $('a[node-type=ytnvWY7q]');
            var $a_grid = $('a[node-type='+wordMap['grid-switch']+']');
            $a_grid.click(function(){
                list_grid_status = 'grid';
            });
        }

        //文件选择框
        function registerCheckbox(){
            //var $checkbox = $('span.checkbox');
            //var $checkbox = $('span.EOGexf');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    var filename;
                    if(list_grid_status == 'list') {
                        //filename = $('div.file-name div.text a',$parent).attr('title');
                        filename = $('div.file-name div.text a',$parent).attr('title');
                    }else if(list_grid_status == 'grid'){
                        //filename = $('div.file-name a',$parent).attr('title');
                        filename = $('div.file-name a',$parent).attr('title');
                    }
                    //if($parent.hasClass('item-active')){
                    //if($parent.hasClass('prWzXA')){
                    if($parent.hasClass(wordMap['item-active'])){
                        slog('取消选中文件：'+filename);
                        for(var i=0;i<selectFileList.length;i++){
                            if(selectFileList[i].filename == filename){
                                selectFileList.splice(i,1);
                            }
                        }
                    }else{
                        slog('选中文件:'+filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterCheckbox(){
            //var $checkbox = $('span.checkbox');
            //var $checkbox = $('span.EOGexf');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //全选框
        function registerAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            //var $checkbox = $('div.Qxyfvg.fydGNC');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    //if($parent.hasClass('checked')){
                    //if($parent.hasClass('EzubGg')){
                    if($parent.hasClass(wordMap['checked'])){
                        slog('取消全选');
                        selectFileList = [];
                    } else {
                        slog('全部选中');
                        selectFileList = [];
                        $.each(fileList,function(index,element){
                            var obj = {
                                filename:element.server_filename,
                                path:element.path,
                                fs_id:element.fs_id,
                                isdir:element.isdir
                            };
                            selectFileList.push(obj);
                        });
                    }
                });
            });
        }

        function unregisterAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            //var $checkbox = $('div.Qxyfvg.fydGNC');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //单个文件选中，点击文件不是点击选中框，会只选中该文件
        function registerFileSelect(){
            //var $dd = $('div.list-view dd');
            //var $dd = $('div.vdAfKMb dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).bind('click',function(e){
                    var nodeName = e.target.nodeName.toLowerCase();
                    if(nodeName != 'span' && nodeName != 'a' && nodeName != 'em') {
                        slog('shiftKey:'+e.shiftKey);
                        if(!e.shiftKey){
                            selectFileList = [];
                            var filename = $('div.file-name div.text a',$(this)).attr('title');
                            slog('选中文件：' + filename);
                            $.each(fileList,function(index,element){
                                if(element.server_filename == filename){
                                    var obj = {
                                        filename:element.server_filename,
                                        path:element.path,
                                        fs_id:element.fs_id,
                                        isdir:element.isdir
                                    };
                                    selectFileList.push(obj);
                                }
                            });
                        }else{
                            selectFileList = [];
                            //var $dd_select = $('div.list-view dd.item-active');
                            //var $dd_select = $('div.vdAfKMb dd.prWzXA');
                            var $dd_select = $('div.'+wordMap['list-view']+' dd.'+wordMap['item-active']);
                            $.each($dd_select,function(index,element){
                                var filename = $('div.file-name div.text a',$(element)).attr('title');
                                slog('选中文件：' + filename);
                                $.each(fileList,function(index,element){
                                    if(element.server_filename == filename){
                                        var obj = {
                                            filename:element.server_filename,
                                            path:element.path,
                                            fs_id:element.fs_id,
                                            isdir:element.isdir
                                        };
                                        selectFileList.push(obj);
                                    }
                                });
                            });
                        }
                    }
                });
            });
        }

        function unregisterFileSelect(){
            //var $dd = $('div.list-view dd');
            //var $dd = $('div.vdAfKMb dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视文件列表显示变化
        function createObserver(){
            var MutationObserver = window.MutationObserver;
            var options = {
                'childList': true
            };
            observer = new MutationObserver(function(mutations){
                unregisterCheckbox();
                unregisterAllCheckbox();
                unregisterFileSelect();
                registerCheckbox();
                registerAllCheckbox();
                registerFileSelect();
            });
            
            //var list_view = document.querySelector('.list-view');
            //var grid_view = document.querySelector('.grid-view');
            
            //var list_view = document.querySelector('.vdAfKMb');
            //var grid_view = document.querySelector('.JKvHJMb');
            
            var list_view = document.querySelector('.'+wordMap['list-view']);
            var grid_view = document.querySelector('.'+wordMap['grid-view']);

            observer.observe(list_view,options);
            observer.observe(grid_view,options);
        }

        //添加助手按钮
        function addButton(){
            //$('div.bar-search').css('width','18%');//修改搜索框的宽度，避免遮挡
            //$('div.OFaPaO').css('width','18%');
            $('div.'+wordMap['bar-search']).css('width','18%');
            var $dropdownbutton = $('<span class="g-dropdown-button"></span>');
            var $dropdownbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><em class="icon icon-download" title="百度网盘下载助手"></em><span class="text" style="width: auto;">下载助手</span></span></a>');
            var $dropdownbutton_span = $('<span class="menu" style="width:96px"></span>');

            var $directbutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $directbutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $directbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">直接下载</span></span></a>');
            var $directbutton_menu = $('<span class="menu" style="width:120px;left:79px"></span>');
            var $directbutton_download_button = $('<a id="download-direct" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $directbutton_link_button = $('<a id="link-direct" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $directbutton_batchhttplink_button = $('<a id="batchhttplink-direct" class="g-button-menu" href="javascript:void(0);">批量链接(HTTP)</a>');
            var $directbutton_batchhttpslink_button = $('<a id="batchhttpslink-direct" class="g-button-menu" href="javascript:void(0);">批量链接(HTTPS)</a>');
            $directbutton_menu.append($directbutton_download_button).append($directbutton_link_button).append($directbutton_batchhttplink_button).append($directbutton_batchhttpslink_button);
            $directbutton.append($directbutton_span.append($directbutton_a).append($directbutton_menu));
            $directbutton.hover(function(){
                $directbutton_span.toggleClass('button-open');
            });
            $directbutton_download_button.click(downloadClick);
            $directbutton_link_button.click(linkClick);
            $directbutton_batchhttplink_button.click(batchClick);
            $directbutton_batchhttpslink_button.click(batchClick);

            var $apibutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $apibutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $apibutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">API下载</span></span></a>');
            var $apibutton_menu = $('<span class="menu" style="width:120px;left:77px"></span>');
            var $apibutton_download_button = $('<a id="download-api" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $apibutton_link_button = $('<a id="httplink-api" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $apibutton_batchhttplink_button = $('<a id="batchhttplink-api" class="g-button-menu" href="javascript:void(0);">批量链接(HTTP)</a>');
            var $apibutton_batchhttpslink_button = $('<a id="batchhttpslink-api" class="g-button-menu" href="javascript:void(0);">批量链接(HTTPS)</a>');
            $apibutton_menu.append($apibutton_download_button).append($apibutton_link_button).append($apibutton_batchhttplink_button).append($apibutton_batchhttpslink_button);
            $apibutton.append($apibutton_span.append($apibutton_a).append($apibutton_menu));
            $apibutton.hover(function(){
                $apibutton_span.toggleClass('button-open');
            });
            $apibutton_download_button.click(downloadClick);
            $apibutton_link_button.click(linkClick);
            $apibutton_batchhttplink_button.click(batchClick);
            $apibutton_batchhttpslink_button.click(batchClick);

            var $outerlinkbutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $outerlinkbutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $outerlinkbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">外链下载</span></span></a>');
            var $outerlinkbutton_menu = $('<span class="menu" style="width:120px;left:79px"></span>');
            var $outerlinkbutton_download_button = $('<a id="download-outerlink" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $outerlinkbutton_link_button = $('<a id="link-outerlink" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $outerlinkbutton_batchlink_button = $('<a id="batchlink-outerlink" class="g-button-menu" href="javascript:void(0);">批量链接</a>');
            $outerlinkbutton_menu.append($outerlinkbutton_download_button).append($outerlinkbutton_link_button).append($outerlinkbutton_batchlink_button);
            $outerlinkbutton.append($outerlinkbutton_span.append($outerlinkbutton_a).append($outerlinkbutton_menu));
            $outerlinkbutton.hover(function(){
                $outerlinkbutton_span.toggleClass('button-open');
            });
            $outerlinkbutton_download_button.click(downloadClick);
            $outerlinkbutton_link_button.click(linkClick);
            $outerlinkbutton_batchlink_button.click(batchClick);

            $dropdownbutton_span.append($directbutton).append($apibutton).append($outerlinkbutton);
            $dropdownbutton.append($dropdownbutton_a).append($dropdownbutton_span);

            $dropdownbutton.hover(function(){
                $dropdownbutton.toggleClass('button-open');
            });

            //$('div.default-dom div.bar div.list-tools').append($dropdownbutton);
            //$('div.irhW9pZ div.yqgR747 div.QDDOQB').append($dropdownbutton);
            $('div.'+wordMap['default-dom']+' div.'+wordMap['bar']+' div.'+wordMap['list-tools']).append($dropdownbutton);
        }

        //暂时没用
        // function addLoading(){
        //     var screenWidth = document.body.clientWidth;
        //     var screenHeight = document.body.scrollHeight;
        //     var left = (screenWidth-10)/2;
        //     var top = screenHeight/2;
        //     var $loading = $('<div id="dialog-loading" style="position:absolute;left:'+left+'px;top:'+top+'px;display:none;z-index:52;color:white;font-size:16px">处理中</div>');
        //     $('body').append($loading);
        // }

        function downloadClick(event){
            slog('选中文件列表：',selectFileList);
            var id = event.target.id;
            var downloadLink;

            if(id == 'download-direct'){
                var downloadType;
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length == 1) {
                    if (selectFileList[0].isdir === 1)
                        downloadType = 'batch';
                    else if (selectFileList[0].isdir === 0)
                        downloadType= 'dlink';
                    //downloadType = selectFileList[0].isdir==1?'batch':(selectFileList[0].isdir===0?'dlink':'batch');
                } else if(selectFileList.length > 1){
                    downloadType = 'batch';
                }
                fid_list = getFidList(selectFileList);
                var result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno === 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch'){
                        downloadLink = result.dlink;
                        if(selectFileList.length === 1)
                            downloadLink = downloadLink + '&zipname=' + encodeURIComponent(selectFileList[0].filename) + '.zip';
                    }
                    else{
                        alert("发生错误！");
                        return;
                    }
                } else if(result.errno == -1){
                    alert('文件不存在或已被百度和谐，无法下载！');
                    return;
                }else if(result.errno == 112){
                    alert("页面过期，请刷新重试！");
                    return;
                }else{
                    alert("发生错误！");
                    return;
                }
            }else{
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length > 1) {
                    alert("该方法不支持多文件下载！");
                    return;
                } else {
                    if(selectFileList[0].isdir == 1){
                        alert("该方法不支持目录下载！");
                        return;
                    }
                }
                if(id == 'download-api'){
                    downloadLink = getDownloadLinkWithRESTAPIBaidu(selectFileList[0].path);
                } else if (id == 'download-outerlink'){
                    var result = getDownloadLinkWithClientAPI(selectFileList[0].path);
                    if(result.errno == 0){
                        downloadLink = result.urls[0].url;
                    }else if(result.errno == 1){
                        alert('文件不存在！');
                        return;
                    }else if(result.errno == 2){
                        alert('文件不存在或者已被百度和谐，无法下载！');
                        return;
                    }else{
                        alert('发生错误！');
                        return;
                    }
                }
            }
            execDownload(downloadLink);
        }

        function linkClick(event){
            slog('选中文件列表：',selectFileList);
            var id = event.target.id;
            var linkList,tip;

            if(id.indexOf('direct') != -1){
                var downloadType;
                var downloadLink;
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length == 1) {
                    if (selectFileList[0].isdir === 1)
                        downloadType = 'batch';
                    else if (selectFileList[0].isdir === 0)
                        downloadType= 'dlink';
                } else if(selectFileList.length > 1){
                    downloadType = 'batch';
                }
                fid_list = getFidList(selectFileList);
                var result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno === 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch'){
                        slog(selectFileList);
                        downloadLink = result.dlink;
                        if(selectFileList.length === 1)
                            downloadLink = downloadLink + '&zipname=' + encodeURIComponent(selectFileList[0].filename) + '.zip';
                    }
                    else{
                        alert("发生错误！");
                        return;
                    }
                }else if(result.errno == -1){
                    alert('文件不存在或已被百度和谐，无法下载！');
                    return;
                }else if(result.errno == 112){
                    alert("页面过期，请刷新重试！");
                    return;
                }else{
                    alert("发生错误！");
                    return;
                }
                var httplink = downloadLink.replace(/^([A-Za-z]+):/,'http:');
                var httpslink = downloadLink.replace(/^([A-Za-z]+):/,'https:');
                var filename = '';
                $.each(selectFileList,function(index,element){
                    if(selectFileList.length == 1)
                        filename = element.filename;
                    else{
                        if(index ==0)
                            filename = element.filename;
                        else
                            filename = filename + ',' + element.filename;
                    }
                });
                linkList = {
                    filename:filename,
                    urls:[
                        {url:httplink,rank:1},
                        {url:httpslink,rank:2}
                    ]
                };
                tip = '显示模拟百度网盘网页获取的链接，可以使用右键迅雷下载，复制到下载工具需要传递cookie，多文件打包下载的链接可以直接复制使用';
                dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
            }else{
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length > 1) {
                    alert("该方法不支持多文件下载！");
                    return;
                } else {
                    if(selectFileList[0].isdir == 1){
                        alert("该方法不支持目录下载！");
                        return;
                    }
                }
                if(id.indexOf('api') != -1){
                    var downloadLink = getDownloadLinkWithRESTAPIBaidu(selectFileList[0].path);
                    var httplink = downloadLink.replace(/^([A-Za-z]+):/,'http:');
                    var httpslink = downloadLink.replace(/^([A-Za-z]+):/,'https:');
                    linkList = {
                        filename:selectFileList[0].filename,
                        urls:[
                            {url:httplink,rank:1},
                            {url:httpslink,rank:2}
                        ]
                    };
                    httplink = httplink.replace('250528','266719');
                    httpslink = httpslink.replace('250528','266719');
                    linkList.urls.push({url:httplink,rank:3});
                    linkList.urls.push({url:httpslink,rank:4});
                    tip = '显示模拟APP获取的链接(使用百度云ID)，可以使用右键迅雷下载，复制到下载工具需要传递cookie';
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
                } else if (id.indexOf('outerlink') != -1){
                    var result = getDownloadLinkWithClientAPI(selectFileList[0].path);
                    if(result.errno == 0){
                        linkList = {
                            filename:selectFileList[0].filename,
                            urls:result.urls
                        };
                    }else if(result.errno == 1){
                        alert('文件不存在！');
                        return;
                    }else if(result.errno == 2){
                        alert('文件不存在或者已被百度和谐，无法下载！');
                        return;
                    }else{
                        alert('发生错误！');
                        return;
                    }
                    tip = '显示模拟百度网盘客户端获取的链接，可以直接复制到下载工具使用，不需要cookie';
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip,showcopy:true,showedit:true});
                }
            }
            //dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
        }

        function batchClick(event){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('获取选中文件失败，请刷新重试！');
                return;
            }
            var id = event.target.id;
            var linkType,tip;
            linkType = id.indexOf('https') == -1 ? (id.indexOf('http') == -1 ? location.protocol+':' : 'http:') : 'https:';
            batchLinkList = [];
            batchLinkListAll = [];
            if(id.indexOf('direct') != -1){
                batchLinkList = getDirectBatchLink(linkType);
                tip = '显示所有选中文件的直接下载链接，文件夹显示为打包下载的链接';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }
                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
            } else if(id.indexOf('api') != -1){
                batchLinkList = getAPIBatchLink(linkType);
                tip = '显示所有选中文件的API下载链接，不显示文件夹';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }
                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
            } else if(id.indexOf('outerlink') != -1){
                batchLinkListAll = getOuterlinkBatchLinkAll();
                batchLinkList = getOuterlinkBatchLinkFirst(batchLinkListAll);
                tip = '显示所有选中文件的外部下载链接，不显示文件夹';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }

                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true,alllist:batchLinkListAll,showall:true});
            }

            //dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
        }

        function getDirectBatchLink(linkType){
            var list = [];
            $.each(selectFileList,function(index,element){
                var downloadType,downloadLink,result;
                if(element.isdir == 0)
                    downloadType = 'dlink';
                else
                    downloadType = 'batch';
                fid_list = getFidList([element]);
                result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno == 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch')
                        downloadLink = result.dlink;
                    downloadLink = downloadLink.replace(/^([A-Za-z]+):/,linkType);
                }else{
                    downloadLink = 'error';
                }
                list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getAPIBatchLink(linkType){
            var list = [];
            $.each(selectFileList,function(index,element){
                if(element.isdir == 1)
                    return;
                var downloadLink;
                downloadLink = getDownloadLinkWithRESTAPIBaidu(element.path);
                downloadLink = downloadLink.replace(/^([A-Za-z]+):/,linkType);
                list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getOuterlinkBatchLinkAll(){
            var list = [];
            $.each(selectFileList,function(index,element){
                var result;
                if(element.isdir == 1)
                    return;
                result = getDownloadLinkWithClientAPI(element.path);
                if(result.errno == 0){
                    //downloadLink = result.urls[0].url;
                    list.push({filename:element.filename,links:result.urls});
                }else{
                    //downloadLink = 'error';
                    list.push({filename:element.filename,links:[{rank:1,url:'error'}]});
                }
                //list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getOuterlinkBatchLinkFirst(list){
            var result = [];
            $.each(list,function(index,element){
                result.push({filename:element.filename,downloadlink:element.links[0].url});
            });
            return result;
        }

        function getSign(){
            var signFnc;
            try{
                signFnc = new Function("return " + yunData.sign2)();
            } catch(e){
                throw new Error(e.message);
            }
            return base64Encode(signFnc(yunData.sign5,yunData.sign1));
        }

        //获取当前目录
        function getPath(){
            var hash = location.hash;
            var regx = /(^|&|\/)path=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取分类显示的类别，即地址栏中的type
        function getCategory(){
            var hash = location.hash;
            var regx = /(^|&|\/)type=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        function getSearchKey(){
            var hash = location.hash;
            var regx = /(^|&|\/)key=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取当前页面(list或者category)
        function getCurrentPage(){
            var hash = location.hash;
            return decodeURIComponent(hash.substring(hash.indexOf('#')+1,hash.indexOf('/')));
        }

        //获取文件列表
        function getFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + "list";
            var path = getPath();
            logid = getLogID();
            var params = {
                dir:path,
                bdstoken:bdstoken,
                logid:logid,
                order:'size',
                desc:0,
                clienttype:0,
                showempty:0,
                web:1,
                channel:'chunlei',
                appid:250528
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.list : [];
                }
            });
            return filelist;
        }

        //获取分类页面下的文件列表
        function getCategoryFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + "categorylist";
            var category = getCategory();
            logid = getLogID();
            var params = {
                category:category,
                bdstoken:bdstoken,
                logid:logid,
                order:'size',
                desc:0,
                clienttype:0,
                showempty:0,
                web:1,
                channel:'chunlei',
                appid:250528
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.info : [];
                }
            });
            return filelist;
        }

        function getSearchFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + 'search';
            logid = getLogID();
            searchKey = getSearchKey();
            var params = {
                recursion:1,
                order:'time',
                desc:1,
                showempty:0,
                web:1,
                page:1,
                num:100,
                key:searchKey,
                channel:'chunlei',
                app_id:250528,
                bdstoken:bdstoken,
                logid:logid,
                clienttype:0
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.list : [];
                }
            });
            return filelist;
        }

        //生成下载时的fid_list参数
        function getFidList(list){
            var fidlist = null;
            if (list.length === 0)
                return null;
            var fileidlist = [];
            $.each(list,function(index,element){
                fileidlist.push(element.fs_id);
            });
            fidlist = '[' + fileidlist + ']';
            return fidlist;
        }

        function getTimestamp(){
            return yunData.timestamp;
        }

        function getBDStoken(){
            return yunData.MYBDSTOKEN;
        }

        //获取直接下载地址
        //这个地址不是直接下载地址，访问这个地址会返回302，response header中的location才是真实下载地址
        //暂时没有找到提取方法
        function getDownloadLinkWithPanAPI(type){
            var downloadUrl = panAPIUrl + "download";
            var result;
            logid = getLogID();
            var params= {
                sign:sign,
                timestamp:timestamp,
                fidlist:fid_list,
                type:type,
                channel:'chunlei',
                web:1,
                app_id:250528,
                bdstoken:bdstoken,
                logid:logid,
                clienttype:0
            };
            $.ajax({
                url:downloadUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    result = response;
                }
            });
            return result;
        }

        function getDownloadLinkWithRESTAPIBaidu(path){
            var link = restAPIUrl + 'file?method=download&app_id=250528&path=' + encodeURIComponent(path);
            return link;
        }

        function getDownloadLinkWithRESTAPIES(path){
            var link = restAPIUrl + 'file?method=download&app_id=266719&path=' + encodeURIComponent(path);
            return link;
        }

        function getDownloadLinkWithClientAPI(path){
            var result;
            var url = clientAPIUrl + 'file?method=locatedownload&app_id=250528&ver=4.0&path=' + encodeURIComponent(path);
            $.ajax({
                url:url,
                method:'POST',
                xhrFields: {
                    withCredentials: true
                },
                async:false,
                success:function(response){
                    result = JSON.parse(response);
                },
                statusCode:{
                    404:function(response){
                        result = response;
                    }
                }
            });
            if(result){
                if(result.error_code == undefined){
                    if(result.urls == undefined){
                        result.errno = 2;
                    }else{
                        $.each(result.urls,function(index,element){
                            result.urls[index].url = element.url.replace('\\','');
                        });
                        result.errno = 0;
                    }
                }else if(result.error_code == 31066){
                    result.errno = 1;
                }else{
                    result.errno = -1;
                }
            }else{
                result = {};
                result.errno = -1;
            }
            return result;
        }

        function execDownload(link){
            slog("下载链接："+link);
            $('#helperdownloadiframe').attr('src',link);
        }

        function createIframe(){
            var $div = $('<div class="helper-hide" style="padding:0;margin:0;display:block"></div>');
            var $iframe = $('<iframe src="javascript:void(0)" id="helperdownloadiframe" style="display:none"></iframe>');
            $div.append($iframe);
            $('body').append($div);

        }
    }

    //分享页面的下载助手
    function PanShareHelper(){
        var yunData,sign,timestamp,bdstoken,channel,clienttype,web,app_id,logid,encrypt,product,uk,primaryid,fid_list,extra,shareid;
        var vcode;
        var shareType,buttonTarget,currentPath,list_grid_status,observer,dialog,vcodeDialog;
        var fileList=[],selectFileList=[];
        var panAPIUrl = location.protocol + "//" + location.host + "/api/";
        var shareListUrl = location.protocol + "//" + location.host + "/share/list";

        this.init = function(){
            yunData = unsafeWindow.yunData;
            slog('yunData:',yunData);
            if(yunData === undefined || yunData.FILEINFO == null){
                slog('页面未正常加载，或者百度已经更新！');
                return;
            }
            initParams();
            addButton();
            dialog = new Dialog({addCopy:false});
            vcodeDialog = new VCodeDialog(refreshVCode,confirmClick);
            createIframe();

            if(!isSingleShare()){
                registerEventListener();
                createObserver();
            }

            slog('分享直接下载加载成功!');
        };

        function initParams(){
            shareType = getShareType();
            sign = yunData.SIGN;
            timestamp = yunData.TIMESTAMP;
            bdstoken = yunData.MYBDSTOKEN;
            channel = 'chunlei';
            clienttype = 0;
            web = 1;
            app_id = 250528;
            logid = getLogID();
            encrypt = 0;
            product = 'share';
            primaryid = yunData.SHARE_ID;
            uk = yunData.SHARE_UK;

            if(shareType == 'secret'){
                extra = getExtra();
            }
            if(isSingleShare()){
                var obj = {};
                if(yunData.CATEGORY == 2){
                    obj.filename = yunData.FILENAME;
                    obj.path = yunData.PATH;
                    obj.fs_id = yunData.FS_ID;
                    obj.isdir = 0;
                } else {
                    obj.filename = yunData.FILEINFO[0].server_filename,
                        obj.path = yunData.FILEINFO[0].path,
                        obj.fs_id = yunData.FILEINFO[0].fs_id,
                        obj.isdir =yunData.FILEINFO[0].isdir
                }
                selectFileList.push(obj);
            } else {
                shareid = yunData.SHARE_ID;
                currentPath = getPath();
                list_grid_status = getListGridStatus();
                fileList = getFileList();
            }
        }

        //判断分享类型（public或者secret）
        function getShareType(){
            return yunData.SHARE_PUBLIC===1 ? 'public' : 'secret';
        }

        //判断是单个文件分享还是文件夹或者多文件分享
        function isSingleShare(){
            return yunData.getContext === undefined ? true : false;
        }

        //判断是否为自己的分享链接
        function isSelfShare(){
            return yunData.MYSELF == 1 ? true : false;
        }

        function getExtra(){
            var seKey = decodeURIComponent(getCookie('BDCLND'));
            return '{' + '"sekey":"' + seKey + '"' + "}";
        }

        //获取当前目录
        function getPath(){
            var hash = location.hash;
            var regx = /(^|&|\/)path=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取当前的视图模式
        function getListGridStatus(){
            var status = 'list';
            var $status_div = $('div.list-grid-switch');
            if ($status_div.hasClass('list-switched-on')){
                status = 'list';
            } else if ($status_div.hasClass('grid-switched-on')) {
                status = 'grid';
            }
            return status;
        }

        //添加下载助手按钮
        function addButton(){
            if(isSingleShare()){
                $('div.slide-show-right').css('width','500px');
                $('div.frame-main').css('width','96%');
                $('div.share-file-viewer').css('width','740px').css('margin-left','auto').css('margin-right','auto');
            }
            else
                $('div.slide-show-right').css('width','500px');
            var $dropdownbutton = $('<span class="g-dropdown-button"></span>');
            var $dropdownbutton_a = $('<a class="g-button" data-button-id="b200" data-button-index="200" href="javascript:void(0);"></a>');
            var $dropdownbutton_a_span = $('<span class="g-button-right"><em class="icon icon-download" title="百度网盘下载助手"></em><span class="text" style="width: auto;">下载助手</span></span>');
            var $dropdownbutton_span = $('<span class="menu" style="width:auto;z-index:41"></span>');

            var $downloadButton = $('<a data-menu-id="b-menu207" class="g-button-menu" href="javascript:void(0);">直接下载</a>');
            var $linkButton = $('<a data-menu-id="b-menu208" class="g-button-menu" href="javascript:void(0);">显示链接</a>');

            $dropdownbutton_span.append($downloadButton).append($linkButton);
            $dropdownbutton_a.append($dropdownbutton_a_span);
            $dropdownbutton.append($dropdownbutton_a).append($dropdownbutton_span);

            $dropdownbutton.hover(function(){
                $dropdownbutton.toggleClass('button-open');
            });

            $downloadButton.click(downloadButtonClick);
            $linkButton.click(linkButtonClick);

            $('div.module-share-top-bar div.bar div.button-box').append($dropdownbutton);
        }

        function createIframe(){
            var $div = $('<div class="helper-hide" style="padding:0;margin:0;display:block"></div>');
            var $iframe = $('<iframe src="javascript:void(0)" id="helperdownloadiframe" style="display:none"></iframe>');
            $div.append($iframe);
            $('body').append($div);
        }

        function registerEventListener(){
            registerHashChange();
            registerListGridStatus();
            registerCheckbox();
            registerAllCheckbox();
            registerFileSelect();
        }

        //监视地址栏#标签变化
        function registerHashChange(){
            window.addEventListener('hashchange',function(e){
                list_grid_status = getListGridStatus();
                if(currentPath == getPath()){
                    return;
                } else {
                    currentPath = getPath();
                    refreshFileList();
                    refreshSelectFileList();
                }
            });
        }

        function refreshFileList(){
            fileList = getFileList();
        }

        function refreshSelectFileList(){
            selectFileList = [];
        }

        //监视视图变化
        function registerListGridStatus(){
            var $a_list = $('a[node-type=list-switch]');
            $a_list.click(function(){
                list_grid_status = 'list';
            });

            var $a_grid = $('a[node-type=grid-switch]');
            $a_grid.click(function(){
                list_grid_status = 'grid';
            });
        }

        //监视文件选择框
        function registerCheckbox(){
            //var $checkbox = $('span.checkbox');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    var filename;
                    if(list_grid_status == 'list') {
                        filename = $('div.file-name div.text a',$parent).attr('title');
                    }else if(list_grid_status == 'grid'){
                        filename = $('div.file-name a',$parent).attr('title');
                    }
                    if($parent.hasClass('item-active')){
                        slog('取消选中文件：'+filename);
                        for(var i=0;i<selectFileList.length;i++){
                            if(selectFileList[i].filename == filename){
                                selectFileList.splice(i,1);
                            }
                        }
                    }else{
                        slog('选中文件：'+filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterCheckbox(){
            //var $checkbox = $('span.checkbox');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视全选框
        function registerAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    //if($parent.hasClass('checked')){
                    if($parent.hasClass(wordMap['checked'])){
                        slog('取消全选');
                        selectFileList = [];
                    } else {
                        slog('全部选中');
                        selectFileList = [];
                        $.each(fileList,function(index,element){
                            var obj = {
                                filename:element.server_filename,
                                path:element.path,
                                fs_id:element.fs_id,
                                isdir:element.isdir
                            };
                            selectFileList.push(obj);
                        });
                    }
                });
            });
        }

        function unregisterAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视单个文件选中
        function registerFileSelect(){
            //var $dd = $('div.list-view dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).bind('click',function(e){
                    var nodeName = e.target.nodeName.toLowerCase();
                    if(nodeName != 'span' && nodeName != 'a' && nodeName != 'em') {
                        selectFileList = [];
                        var filename = $('div.file-name div.text a',$(this)).attr('title');
                        slog('选中文件：' + filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterFileSelect(){
            //var $dd = $('div.list-view dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视文件列表显示变化
        function createObserver(){
            var MutationObserver = window.MutationObserver;
            var options = {
                'childList': true
            };
            observer = new MutationObserver(function(mutations){
                unregisterCheckbox();
                unregisterAllCheckbox();
                unregisterFileSelect();
                registerCheckbox();
                registerAllCheckbox();
                registerFileSelect(); 
            });
            //var list_view = document.querySelector('.list-view');
            //var grid_view = document.querySelector('.grid-view');
            
            var list_view = document.querySelector('.'+wordMap['list-view']);
            var grid_view = document.querySelector('.'+wordMap['grid-view']);

            observer.observe(list_view,options);
            observer.observe(grid_view,options);
        }

        //获取文件信息列表
        function getFileList(){
            var result = [];
            if(getPath() == '/'){
                result = yunData.FILEINFO;
            } else {
                logid = getLogID();
                var params = {
                    uk:uk,
                    shareid:shareid,
                    order:'other',
                    desc:1,
                    showempty:0,
                    web:web,
                    dir:getPath(),
                    t:Math.random(),
                    bdstoken:bdstoken,
                    channel:channel,
                    clienttype:clienttype,
                    app_id:app_id,
                    logid:logid
                };
                $.ajax({
                    url:shareListUrl,
                    method:'GET',
                    async:false,
                    data:params,
                    success:function(response){
                        if(response.errno === 0){
                            result = response.list;
                        }
                    }
                });
            }
            return result;
        }

        function downloadButtonClick(){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('获取文件ID失败，请重试');
                return;
            }
            buttonTarget = 'download';
            var downloadLink = getDownloadLink();

            if(downloadLink.errno == -20) {
                vcode = getVCode();
                if(vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open(vcode);
            } else if(downloadLink.errno == 112){
                alert('页面过期，请刷新重试');
                return;
            } else if (downloadLink.errno === 0) {
                var link;
                if(selectFileList.length == 1 && selectFileList[0].isdir === 0)
                    link = downloadLink.list[0].dlink;
                else
                    link = downloadLink.dlink;
                execDownload(link);
            } else {
                alert('获取下载链接失败！');
                return;
            }
        }

        //获取验证码
        function getVCode(){
            var url = panAPIUrl + 'getvcode';
            var result;
            logid = getLogID();
            var params = {
                prod:'pan',
                t:Math.random(),
                bdstoken:bdstoken,
                channel:channel,
                clienttype:clienttype,
                web:web,
                app_id:app_id,
                logid:logid
            };
            $.ajax({
                url:url,
                method:'GET',
                async:false,
                data:params,
                success:function(response){
                    result = response;
                }
            });
            return result;
        }

        //刷新验证码
        function refreshVCode(){
            vcode = getVCode();
            $('#dialog-img').attr('src',vcode.img);
        }

        //验证码确认提交
        function confirmClick(){
            var val = $('#dialog-input').val();
            if(val.length === 0) {
                $('#dialog-err').text('请输入验证码');
                return;
            } else if(val.length < 4) {
                $('#dialog-err').text('验证码输入错误，请重新输入');
                return;
            } 
            var result = getDownloadLinkWithVCode(val);
            if(result.errno == -20){
                vcodeDialog.close();
                $('#dialog-err').text('验证码输入错误，请重新输入');
                refreshVCode();
                if(!vcode || vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open();
            } else if (result.errno === 0) {
                vcodeDialog.close();
                var link;
                if(selectFileList.length ==1 && selectFileList[0].isdir === 0)
                    link = result.list[0].dlink;
                else
                    link = result.dlink;
                if(buttonTarget == 'download'){
                    execDownload(link);
                } else if (buttonTarget == 'link') {
                    var filename = '';
                    $.each(selectFileList,function(index,element){
                        if(selectFileList.length == 1)
                            filename = element.filename;
                        else{
                            if(index ==0)
                                filename = element.filename;
                            else
                                filename = filename + ',' + element.filename;
                        }
                    });
                    var linkList = {
                        filename:filename,
                        urls:[
                            {url:link,rank:1}
                        ]
                    };
                    var tip = "显示获取的链接，可以使用右键迅雷下载，复制无用，需要传递cookie";
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
                }
            } else {
                alert('发生错误！');
                return;
            }
        }

        //生成下载用的fid_list参数
        function getFidList(){
            var fidlist = [];
            $.each(selectFileList,function(index,element){
                fidlist.push(element.fs_id);
            });
            return '[' + fidlist + ']';
        }

        function linkButtonClick(){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('没有选中文件，请重试');
                return;
            }
            buttonTarget = 'link';
            var downloadLink = getDownloadLink();

            if(downloadLink.errno == -20) {
                vcode = getVCode();
                if(!vcode || vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open(vcode);
            } else if (downloadLink.errno == 112) {
                alert('页面过期，请刷新重试');
                return;
            } else if (downloadLink.errno === 0) {
                var link;
                if(selectFileList.length == 1 && selectFileList[0].isdir === 0)
                    link = downloadLink.list[0].dlink;
                else
                    link = downloadLink.dlink;
                if(selectFileList.length == 1)
                    $('#dialog-downloadlink').attr('href',link).text(link);
                else
                    $('#dialog-downloadlink').attr('href',link).text(link);
                var filename = '';
                $.each(selectFileList,function(index,element){
                    if(selectFileList.length == 1)
                        filename = element.filename;
                    else{
                        if(index ==0)
                            filename = element.filename;
                        else
                            filename = filename + ',' + element.filename;
                    }
                });
                var linkList = {
                    filename:filename,
                    urls:[
                        {url:link,rank:1}
                    ]
                };
                var tip = "显示获取的链接，可以使用右键迅雷下载，复制无用，需要传递cookie";
                dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
            } else {
                alert('获取下载链接失败！');
                return;
            }
        }

        //获取下载链接
        function getDownloadLink(){
            var result;
            if(isSingleShare){
                fid_list = getFidList();
                logid = getLogID();
                var url = panAPIUrl + 'sharedownload?sign=' + sign + '&timestamp=' + timestamp + '&bdstoken=' + bdstoken + '&channel=' + channel + '&clienttype=' + clienttype + '&web='+ web + '&app_id=' + app_id + '&logid=' + logid;
                var params = {
                    encrypt:encrypt,
                    product:product,
                    uk:uk,
                    primaryid:primaryid,
                    fid_list:fid_list
                };
                if(shareType == 'secret'){
                    params.extra = extra;
                }
                if(selectFileList[0].isdir == 1 || selectFileList.length > 1){
                    params.type = 'batch';
                }
                $.ajax({
                    url:url,
                    method:'POST',
                    async:false,
                    data:params,
                    success:function(response){
                        result = response;
                    }
                });
            }
            return result;
        }

        //有验证码输入时获取下载链接
        function getDownloadLinkWithVCode(vcodeInput){
            var result;
            if(isSingleShare){
                fid_list = getFidList();
                var url = panAPIUrl + 'sharedownload?sign=' + sign + '&timestamp=' + timestamp + '&bdstoken=' + bdstoken + '&channel=' + channel + '&clienttype=' + clienttype + '&web='+ web + '&app_id=' + app_id + '&logid=' + logid;
                var params = {
                    encrypt:encrypt,
                    product:product,
                    vcode_input:vcodeInput,
                    vcode_str:vcode.vcode,
                    uk:uk,
                    primaryid:primaryid,
                    fid_list:fid_list
                };
                if(shareType == 'secret'){
                    params.extra = extra;
                }
                if(selectFileList[0].isdir == 1 || selectFileList.length > 1 ){
                    params.type = 'batch';
                }
                $.ajax({
                    url:url,
                    method:'POST',
                    async:false,
                    data:params,
                    success:function(response){
                        result = response;
                    }
                });
            }
            return result;
        }

        function execDownload(link){
            slog('下载链接：'+link);
            $('#helperdownloadiframe').attr('src',link);
        }
    }

    function base64Encode(t){
        var a, r, e, n, i, s, o = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
        for (e = t.length,r = 0,a = ""; e > r; ) {
            if (n = 255 & t.charCodeAt(r++),r == e) {
                a += o.charAt(n >> 2);
                a += o.charAt((3 & n) << 4);
                a += "==";
                break;
            }
            if (i = t.charCodeAt(r++),r == e) {
                a += o.charAt(n >> 2);
                a += o.charAt((3 & n) << 4 | (240 & i) >> 4);
                a += o.charAt((15 & i) << 2);
                a += "=";
                break;
            }
            s = t.charCodeAt(r++);
            a += o.charAt(n >> 2);
            a += o.charAt((3 & n) << 4 | (240 & i) >> 4);
            a += o.charAt((15 & i) << 2 | (192 & s) >> 6);
            a += o.charAt(63 & s);
        }
        return a;
    }

    function detectPage(){
        var regx = /[\/].+[\/]/g;
        var page = location.pathname.match(regx);
        return page[0].replace(/\//g,'');
    }

    function getCookie(e) {
        var o, t;
        var n = document,c=decodeURI;
        return n.cookie.length > 0 && (o = n.cookie.indexOf(e + "="),-1 != o) ? (o = o + e.length + 1,t = n.cookie.indexOf(";", o),-1 == t && (t = n.cookie.length),c(n.cookie.substring(o, t))) : "";
    }

    function getLogID(){
        var name = "BAIDUID";
        var u = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/~！@#￥%……&";
        var d = /[\uD800-\uDBFF][\uDC00-\uDFFFF]|[^\x00-\x7F]/g;
        var f = String.fromCharCode;
        function l(e){
            if (e.length < 2) {
                var n = e.charCodeAt(0);
                return 128 > n ? e : 2048 > n ? f(192 | n >>> 6) + f(128 | 63 & n) : f(224 | n >>> 12 & 15) + f(128 | n >>> 6 & 63) + f(128 | 63 & n);
            }
            var n = 65536 + 1024 * (e.charCodeAt(0) - 55296) + (e.charCodeAt(1) - 56320);
            return f(240 | n >>> 18 & 7) + f(128 | n >>> 12 & 63) + f(128 | n >>> 6 & 63) + f(128 | 63 & n);
        }
        function g(e){
            return (e + "" + Math.random()).replace(d, l);
        }
        function m(e){
            var n = [0, 2, 1][e.length % 3];
            var t = e.charCodeAt(0) << 16 | (e.length > 1 ? e.charCodeAt(1) : 0) << 8 | (e.length > 2 ? e.charCodeAt(2) : 0);
            var o = [u.charAt(t >>> 18), u.charAt(t >>> 12 & 63), n >= 2 ? "=" : u.charAt(t >>> 6 & 63), n >= 1 ? "=" : u.charAt(63 & t)];
            return o.join("");
        }
        function h(e){
            return e.replace(/[\s\S]{1,3}/g, m);
        }
        function p(){
            return h(g((new Date()).getTime()));
        }
        function w(e,n){
            return n ? p(String(e)).replace(/[+\/]/g, function(e) {
                return "+" == e ? "-" : "_";
            }).replace(/=/g, "") : p(String(e));
        }
        return w(getCookie(name));
    }

    function Dialog(){
        var linkList = [];
        var showParams;
        var dialog,shadow;
        function createDialog(){
            var screenWidth = document.body.clientWidth;
            var dialogLeft = screenWidth>800 ? (screenWidth-800)/2 : 0;
            var $dialog_div = $('<div class="dialog" style="width: 800px; top: 0px; bottom: auto; left: '+dialogLeft+'px; right: auto; display: hidden; visibility: visible; z-index: 52;"></div>');
            var $dialog_header = $('<div class="dialog-header"><h3><span class="dialog-title" style="display:inline-block;width:740px;white-space:nowrap;overflow-x:hidden;text-overflow:ellipsis"></span></h3></div>');
            var $dialog_control = $('<div class="dialog-control"><span class="dialog-icon dialog-close">×</span></div>');
            var $dialog_body = $('<div class="dialog-body" style="max-height:450px;overflow-y:auto;padding:0 20px;"></div>');
            var $dialog_tip = $('<div class="dialog-tip" style="padding-left:20px;background-color:#faf2d3;border-top: 1px solid #c4dbfe;"><p></p></div>');

            $dialog_div.append($dialog_header.append($dialog_control)).append($dialog_body);

            //var $dialog_textarea = $('<textarea class="dialog-textarea" style="display:none;width"></textarea>');
            var $dialog_radio_div = $('<div class="dialog-radio" style="display:none;width:760px;padding-left:20px;padding-right:20px"></div>');
            var $dialog_radio_multi = $('<input type="radio" name="showmode" checked="checked" value="multi"><span>多行</span>');
            var $dialog_radio_single = $('<input type="radio" name="showmode" value="single"><span>单行</span>');
            $dialog_radio_div.append($dialog_radio_multi).append($dialog_radio_single);
            $dialog_div.append($dialog_radio_div);
            $('input[type=radio][name=showmode]',$dialog_radio_div).change(function(){
                var value = this.value;
                var $textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var content = $textarea.val();
                if(value == 'multi'){
                    content = content.replace(/\s+/g,'\n');
                    $textarea.css('height','300px');
                } else if(value == 'single'){
                    content = content.replace(/\n+/g,' ');
                    $textarea.css('height','');
                }
                $textarea.val(content);
            });

            var $dialog_button = $('<div class="dialog-button" style="display:none"></div>');
            var $dialog_button_div = $('<div style="display:table;margin:auto"></div>')
            var $dialog_copy_button = $('<button id="dialog-copy-button" style="display:none">复制</button>');
            var $dialog_edit_button = $('<button id="dialog-edit-button" style="display:none">编辑</button>');
            var $dialog_exit_button = $('<button id="dialog-exit-button" style="display:none">退出</button>');

            $dialog_button_div.append($dialog_copy_button).append($dialog_edit_button).append($dialog_exit_button);
            $dialog_button.append($dialog_button_div);
            $dialog_div.append($dialog_button);

            $dialog_copy_button.click(function(){
                var content = '';
                if(showParams.type == 'batch'){
                    $.each(linkList,function(index,element){
                        if(element.downloadlink == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.downloadlink;
                        else
                            content =  content + element.downloadlink + '\n';
                    });
                } else if(showParams.type == 'link'){
                    $.each(linkList,function(index,element){
                        if(element.url == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.url;
                        else
                            content =  content + element.url + '\n';
                    });
                }
                GM_setClipboard(content,'text');
                alert('已将链接复制到剪贴板！');
            });

            $dialog_edit_button.click(function(){
                var $dialog_textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var $dialog_item = $('div.dialog-body div',dialog);
                $dialog_item.hide();
                $dialog_copy_button.hide();
                $dialog_edit_button.hide();
                $dialog_textarea.show();
                $dialog_radio_div.show();
                $dialog_exit_button.show();
            });

            $dialog_exit_button.click(function(){
                var $dialog_textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var $dialog_item = $('div.dialog-body div',dialog);
                $dialog_textarea.hide();
                $dialog_radio_div.hide();
                $dialog_item.show();
                $dialog_exit_button.hide();
                $dialog_copy_button.show();
                $dialog_edit_button.show();
            });

            $dialog_div.append($dialog_tip);
            $('body').append($dialog_div);
            $dialog_div.dialogDrag();
            $dialog_control.click(dialogControl);
            return $dialog_div;
        }

        function createShadow(){
            var $shadow = $('<div class="dialog-shadow" style="position: fixed; left: 0px; top: 0px; z-index: 50; background: rgb(0, 0, 0) none repeat scroll 0% 0%; opacity: 0.5; width: 100%; height: 100%; display: none;"></div>');
            $('body').append($shadow);
            return $shadow;
        }

        this.open = function(params){
            showParams = params;
            linkList = [];
            if(params.type == 'link'){
                linkList = params.list.urls;
                $('div.dialog-header h3 span.dialog-title',dialog).text(params.title + "：" +params.list.filename);
                $.each(params.list.urls,function(index,element){
                    var $div = $('<div><div style="width:30px;float:left">'+element.rank+':</div><div style="white-space:nowrap;overflow:hidden;text-overflow:ellipsis"><a href="'+element.url+'">'+element.url+'</a></div></div>');
                    $('div.dialog-body',dialog).append($div);
                });
            } else if(params.type == 'batch'){
                linkList = params.list;
                $('div.dialog-header h3 span.dialog-title',dialog).text(params.title);
                if(params.showall){
                    $.each(params.list,function(index,element){
                        var $item_div = $('<div class="item-container" style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap"></div>');
                        var $item_name = $('<div style="width:100px;float:left;overflow:hidden;text-overflow:ellipsis" title="'+element.filename+'">'+element.filename+'</div>');
                        var $item_sep = $('<div style="width:12px;float:left"><span>：</span></div>');
                        var $item_link_div = $('<div class="item-link" style="float:left;width:618px;"></div>');
                        var $item_first = $('<div class="item-first" style="overflow:hidden;text-overflow:ellipsis"><a href="'+element.downloadlink+'">'+element.downloadlink+'</a></div>');
                        $item_link_div.append($item_first);
                        $.each(params.alllist[index].links,function(n,item){
                            if(element.downloadlink == item.url)
                                return;
                            var $item = $('<div class="item-ex" style="display:none;overflow:hidden;text-overflow:ellipsis"><a href="'+item.url+'">'+item.url+'</a></div>');
                            $item_link_div.append($item);
                        });
                        var $item_ex = $('<div style="width:15px;float:left;cursor:pointer;text-align:center;font-size:16px"><span>+</span></div>');
                        $item_div.append($item_name).append($item_sep).append($item_link_div).append($item_ex);
                        $item_ex.click(function(){
                            var $parent = $(this).parent();
                            $parent.toggleClass('showall');
                            if($parent.hasClass('showall')){
                                $(this).text('-');
                                $('div.item-link div.item-ex',$parent).show();
                            } else {
                                $(this).text('+');
                                $('div.item-link div.item-ex',$parent).hide();
                            }
                        });
                        $('div.dialog-body',dialog).append($item_div);
                    });
                }else{
                    $.each(params.list,function(index,element){
                        var $div = $('<div style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap"><div style="width:100px;float:left;overflow:hidden;text-overflow:ellipsis" title="'+element.filename+'">'+element.filename+'</div><span>：</span><a href="'+element.downloadlink+'">'+element.downloadlink+'</a></div>');
                        $('div.dialog-body',dialog).append($div);
                    });
                }
            }

            if(params.tip){
                $('div.dialog-tip p',dialog).text(params.tip);
            }

            if(params.showcopy){
                $('div.dialog-button',dialog).show();
                $('div.dialog-button button#dialog-copy-button',dialog).show();
            }
            if(params.showedit){
                $('div.dialog-button',dialog).show();
                $('div.dialog-button button#dialog-edit-button',dialog).show();
                var $dialog_textarea = $('<textarea name="dialog-textarea" style="display:none;resize:none;width:758px;height:300px;white-space:pre;word-wrap:normal;overflow-x:scroll"></textarea>');
                var content = '';
                if(showParams.type == 'batch'){
                    $.each(linkList,function(index,element){
                        if(element.downloadlink == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.downloadlink;
                        else
                            content =  content + element.downloadlink + '\n';
                    });
                } else if(showParams.type == 'link'){
                    $.each(linkList,function(index,element){
                        if(element.url == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.url;
                        else
                            content =  content + element.url + '\n';
                    });
                }
                $dialog_textarea.val(content);
                $('div.dialog-body',dialog).append($dialog_textarea);
            }

            shadow.show();
            dialog.show();
        }

        this.close = function(){
            dialogControl();
        }

        function dialogControl(){
            $('div.dialog-body',dialog).children().remove();
            $('div.dialog-header h3 span.dialog-title',dialog).text('');
            $('div.dialog-tip p',dialog).text('');
            $('div.dialog-button',dialog).hide();
            $('div.dialog-radio input[type=radio][name=showmode][value=multi]',dialog).prop('checked',true);
            $('div.dialog-radio',dialog).hide();
            $('div.dialog-button button#dialog-copy-button',dialog).hide();
            $('div.dialog-button button#dialog-edit-button',dialog).hide();
            $('div.dialog-button button#dialog-exit-button',dialog).hide();
            dialog.hide();
            shadow.hide();
        }

        dialog = createDialog();
        shadow = createShadow();
    }

    function VCodeDialog(refreshVCode,confirmClick){
        var dialog,shadow;
        function createDialog(){
            var screenWidth = document.body.clientWidth;
            var dialogLeft = screenWidth>520 ? (screenWidth-520)/2 : 0;
            var $dialog_div = $('<div class="dialog" id="dialog-vcode" style="width:520px;top:0px;bottom:auto;left:'+dialogLeft+'px;right:auto;display:none;visibility:visible;z-index:52"></div>');
            var $dialog_header = $('<div class="dialog-header"><h3><span class="dialog-header-title"><em class="select-text">提示</em></span></h3></div>');
            var $dialog_control = $('<div class="dialog-control"><span class="dialog-icon dialog-close icon icon-close"><span class="sicon">x</span></span></div>');
            var $dialog_body = $('<div class="dialog-body"></div>');
            var $dialog_body_div = $('<div style="text-align:center;padding:22px"></div>');
            var $dialog_body_download_verify = $('<div class="download-verify" style="margin-top:10px;padding:0 28px;text-align:left;font-size:12px;"></div>');
            var $dialog_verify_body = $('<div class="verify-body">请输入验证码：</div>');
            var $dialog_input = $('<input id="dialog-input" type="text" style="padding:3px;width:85px;height:23px;border:1px solid #c6c6c6;background-color:white;vertical-align:middle;" class="input-code" maxlength="4">');
            var $dialog_img = $('<img id="dialog-img" class="img-code" style="margin-left:10px;vertical-align:middle;" alt="点击换一张" src="" width="100" height="30">');
            var $dialog_refresh = $('<a href="javascript:void(0)" style="text-decoration:underline;" class="underline">换一张</a>');
            var $dialog_err = $('<div id="dialog-err" style="padding-left:84px;height:18px;color:#d80000" class="verify-error"></div>');
            var $dialog_footer = $('<div class="dialog-footer g-clearfix"></div>');
            var $dialog_confirm_button = $('<a class="g-button g-button-blue" data-button-id="" data-button-index href="javascript:void(0)" style="padding-left:36px"><span class="g-button-right" style="padding-right:36px;"><span class="text" style="width:auto;">确定</span></span></a>');
            var $dialog_cancel_button = $('<a class="g-button" data-button-id="" data-button-index href="javascript:void(0);" style="padding-left: 36px;"><span class="g-button-right" style="padding-right: 36px;"><span class="text" style="width: auto;">取消</span></span></a>');

            $dialog_header.append($dialog_control);
            $dialog_verify_body.append($dialog_input).append($dialog_img).append($dialog_refresh);
            $dialog_body_download_verify.append($dialog_verify_body).append($dialog_err);
            $dialog_body_div.append($dialog_body_download_verify);
            $dialog_body.append($dialog_body_div);
            $dialog_footer.append($dialog_confirm_button).append($dialog_cancel_button);
            $dialog_div.append($dialog_header).append($dialog_body).append($dialog_footer);
            $('body').append($dialog_div);

            $dialog_div.dialogDrag();

            $dialog_control.click(dialogControl);
            $dialog_img.click(refreshVCode);
            $dialog_refresh.click(refreshVCode);
            $dialog_input.keypress(function(event){
                if(event.which == 13)
                    confirmClick();
            });
            $dialog_confirm_button.click(confirmClick);
            $dialog_cancel_button.click(dialogControl);
            $dialog_input.click(function(){
                $('#dialog-err').text('');
            });
            return $dialog_div;
        }
        this.open = function(vcode){
            if(vcode)
                $('#dialog-img').attr('src',vcode.img);
            dialog.show();
            shadow.show();
        }
        this.close = function(){
            dialogControl();
        }
        dialog = createDialog();
        shadow = $('div.dialog-shadow');
        function dialogControl(){
            $('#dialog-img',dialog).attr('src','');
            $('#dialog-err').text('');
            dialog.hide();
            shadow.hide();
        }
    }

    $.fn.dialogDrag = function(){
        var mouseInitX,mouseInitY,dialogInitX,dialogInitY;
        var screenWidth = document.body.clientWidth;
        var $parent = this;
        $('div.dialog-header',this).mousedown(function(event){
            mouseInitX = parseInt(event.pageX);
            mouseInitY = parseInt(event.pageY);
            dialogInitX = parseInt($parent.css('left').replace('px',''));
            dialogInitY = parseInt($parent.css('top').replace('px',''));
            $(this).mousemove(function(event){
                var tempX = dialogInitX + parseInt(event.pageX) - mouseInitX;
                var tempY = dialogInitY + parseInt(event.pageY) - mouseInitY;
                var width = parseInt($parent.css('width').replace('px',''));
                tempX = tempX<0 ? 0 : tempX>screenWidth-width ? screenWidth-width : tempX;
                tempY = tempY<0 ? 0 : tempY;
                $parent.css('left',tempX+'px').css('top',tempY+'px');
            });
        });
        $('div.dialog-header',this).mouseup(function(event){
            $(this).unbind('mousemove');
        });
    }

})();


// ==UserScript==
// @name         解决百度云大文件下载限制
// @namespace    undefined
// @version      0.0.6
// @description  一行代码，解决百度云大文件下载限制
// @author       funianwuxin
// @match        http://pan.baidu.com/*
// @match        https://pan.baidu.com/*
// @match        http://yun.baidu.com/*
// @match        https://yun.baidu.com/*
// @match        https://eyun.baidu.com/*
// @run-at       document-start
// @grant        none
// ==/UserScript==
/* jshint -W097 */
'use strict';

Object.defineProperty(Object.getPrototypeOf(navigator),'platform',{get:function(){return 'sb_baidu';}})


(function(){
var href=location.href;
/http:/.test(href)?location.href='https'+href.slice(4):0;
}());


    // ==UserScript==
// @name              去百度搜索置顶推广 (ECMA6)
// @author            axetroy
// @contributor       axetroy
// @description       去除插入在百度搜索结果头部、尾部的推广链接。
// @version           2016.6.4
// @grant             none
// @include           *www.baidu.com*
// @include           *zhidao.baidu.com/search*
// @connect           *
// @supportURL        http://www.burningall.com
// @compatible        chrome  完美运行
// @compatible        firefox  完美运行
// @run-at            document-start
// @contributionURL   troy450409405@gmail.com|alipay.com
// @namespace         https://greasyfork.org/zh-CN/users/3400-axetroy
// @license           The MIT License (MIT); http://opensource.org/licenses/MIT
// ==/UserScript==

/*

 Github源码:https://github.com/axetroy/GMscript

 */
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};

/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {

/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId])
/******/ 			return installedModules[moduleId].exports;

/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			exports: {},
/******/ 			id: moduleId,
/******/ 			loaded: false
/******/ 		};

/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

/******/ 		// Flag the module as loaded
/******/ 		module.loaded = true;

/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}


/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;

/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;

/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";

/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ function(module, exports, __webpack_require__) {

	'use strict';

	var _jqLite = __webpack_require__(1);

	var _jqLite2 = _interopRequireDefault(_jqLite);

	var _$interval = __webpack_require__(55);

	var _$interval2 = _interopRequireDefault(_$interval);

	var _main = __webpack_require__(56);

	var _main2 = _interopRequireDefault(_main);

	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

	var loop = (0, _$interval2.default)(function () {
	  new _main2.default().filter().turn();
	}, 50);

	// init
	(0, _jqLite2.default)(function () {
	  new _main2.default().filter().turn();
	  _$interval2.default.cancel(loop);
	  (0, _jqLite2.default)(document).observe(function () {
	    return new _main2.default().filter().turn();
	  });
	  console.info('去广告启动...');
	});

/***/ },
/* 1 */
/***/ function(module, exports, __webpack_require__) {

	'use strict';

	// es6 Array.from

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});

	var _typeof = typeof Symbol === "function" && typeof Symbol.iterator === "symbol" ? function (obj) { return typeof obj; } : function (obj) { return obj && typeof Symbol === "function" && obj.constructor === Symbol ? "symbol" : typeof obj; };

	var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();
	// es6 Object.assign


	__webpack_require__(2);

	__webpack_require__(30);

	function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

	var noop = function noop(x) {
	  return x;
	};

	var jqLite = function () {
	  function jqLite() {
	    var _this = this;

	    var selectors = arguments.length <= 0 || arguments[0] === undefined ? '' : arguments[0];
	    var context = arguments.length <= 1 || arguments[1] === undefined ? document : arguments[1];

	    _classCallCheck(this, jqLite);

	    this.selectors = selectors;
	    this.context = context;
	    this.length = 0;

	    switch (typeof selectors === 'undefined' ? 'undefined' : _typeof(selectors)) {
	      case 'undefined':
	        break;
	      case 'string':
	        Array.from(context.querySelectorAll(selectors), function (ele, i) {
	          _this[i] = ele;
	          _this.length++;
	        }, this);
	        break;
	      case 'object':
	        if (selectors.length) {
	          Array.from(selectors, function (ele, i) {
	            _this[i] = ele;
	            _this.length++;
	          }, this);
	        } else {
	          this[0] = selectors;
	          this.length = 1;
	        }
	        break;
	      case 'function':
	        this.ready(selectors);
	        break;
	      default:

	    }
	  }

	  _createClass(jqLite, [{
	    key: 'eq',
	    value: function eq() {
	      var n = arguments.length <= 0 || arguments[0] === undefined ? 0 : arguments[0];

	      return new jqLite(this[n]);
	    }
	  }, {
	    key: 'find',
	    value: function find(selectors) {
	      return new jqLite(selectors, this[0]);
	    }
	  }, {
	    key: 'each',
	    value: function each() {
	      var fn = arguments.length <= 0 || arguments[0] === undefined ? noop : arguments[0];

	      for (var i = 0; i < this.length; i++) {
	        fn.call(this, this[i], i);
	      }
	      return this;
	    }
	  }, {
	    key: 'bind',
	    value: function bind() {
	      var types = arguments.length <= 0 || arguments[0] === undefined ? '' : arguments[0];
	      var fn = arguments.length <= 1 || arguments[1] === undefined ? noop : arguments[1];

	      this.each(function (ele) {
	        types.trim().split(/\s{1,}/).forEach(function (type) {
	          ele.addEventListener(type, function (e) {
	            var target = e.target || e.srcElement;
	            if (fn.call(target, e) === false) {
	              e.returnValue = true;
	              e.cancelBubble = true;
	              e.preventDefault && e.preventDefault();
	              e.stopPropagation && e.stopPropagation();
	              return false;
	            }
	          }, false);
	        });
	      });
	    }
	  }, {
	    key: 'click',
	    value: function click() {
	      var fn = arguments.length <= 0 || arguments[0] === undefined ? noop : arguments[0];

	      this.bind('click', fn);
	      return this;
	    }
	  }, {
	    key: 'ready',
	    value: function ready() {
	      var _this2 = this;

	      var fn = arguments.length <= 0 || arguments[0] === undefined ? noop : arguments[0];

	      window.addEventListener('DOMContentLoaded', function (e) {
	        fn.call(_this2, e);
	      }, false);
	    }
	  }, {
	    key: 'observe',
	    value: function observe() {
	      var _this3 = this;

	      var fn = arguments.length <= 0 || arguments[0] === undefined ? noop : arguments[0];
	      var config = arguments.length <= 1 || arguments[1] === undefined ? { childList: true, subtree: true } : arguments[1];

	      this.each(function (ele) {
	        var MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver;
	        var observer = new MutationObserver(function (mutations) {
	          mutations.forEach(function (mutation) {
	            fn.call(_this3, mutation.target, mutation.addedNodes, mutation.removedNodes);
	          });
	        });
	        observer.observe(ele, config);
	      });
	      return this;
	    }
	  }, {
	    key: 'attr',
	    value: function (_attr) {
	      function attr(_x, _x2) {
	        return _attr.apply(this, arguments);
	      }

	      attr.toString = function () {
	        return _attr.toString();
	      };

	      return attr;
	    }(function (attr, value) {
	      // one agm
	      if (arguments.length === 1) {
	        // get attr value
	        if (typeof attr === 'string') {
	          return this[0].getAttribute(attr);
	        }
	        // set attr with a json
	        else if ((typeof attr === 'undefined' ? 'undefined' : _typeof(attr)) === 'object') {
	            this.each(function (ele) {
	              for (var at in attr) {
	                if (attr.hasOwnProperty(at)) {
	                  ele.setAttribute(at, value);
	                }
	              }
	            });
	            return value;
	          }
	      }
	      // set
	      else if (arguments.length === 2) {
	          this.each(function (ele) {
	            ele.setAttribute(attr, value);
	          });
	          return this;
	        } else {
	          return this;
	        }
	    })
	  }, {
	    key: 'removeAttr',
	    value: function removeAttr(attr) {
	      if (arguments.length === 1) {
	        this.each(function (ele) {
	          ele.removeAttribute(attr);
	        });
	      }
	      return this;
	    }
	  }, {
	    key: 'remove',
	    value: function remove() {
	      this.each(function (ele) {
	        ele.remove();
	      });
	      this.length = 0;
	      return this;
	    }

	    // get the element style

	  }, {
	    key: 'style',
	    value: function style(attr) {
	      return this[0].currentStyle ? this[0].currentStyle[attr] : getComputedStyle(this[0])[attr];
	    }

	    // (attr,value) || 'string' || {}

	  }, {
	    key: 'css',
	    value: function css() {
	      for (var _len = arguments.length, agm = Array(_len), _key = 0; _key < _len; _key++) {
	        agm[_key] = arguments[_key];
	      }

	      if (agm.length === 1) {
	        // get style
	        if (typeof agm[0] === 'string') {
	          // set style as a long text
	          if (/:/ig.test(agm[0])) {
	            this.each(function (ele) {
	              ele.style.cssText = attr;
	            });
	          } else {
	            return this[0].currentStyle ? this[0].currentStyle[agm[0]] : getComputedStyle(this[0])[agm[0]];
	          }
	        }
	        // set style as a object
	        else {
	            this.each(function (ele) {
	              for (var _attr2 in agm[0]) {
	                if (agm[0].hasOwnProperty(_attr2)) {
	                  ele.style[_attr2] = agm[0][_attr2];
	                }
	              }
	            });
	          }
	      }
	      // set as (key,value)
	      else if (agm.length === 2) {
	          this.each(function (ele) {
	            ele.style[agm[0]] = agm[1];
	          });
	        }
	      return this;
	    }
	  }, {
	    key: 'width',
	    value: function width(value) {
	      var element = this[0];
	      // window or document
	      if (element.window === element || element.body) {
	        return document.body.scrollWidth > document.documentElement.scrollWidth ? document.body.scrollWidth : document.documentElement.scrollWidth;
	      }
	      // set width
	      else if (value) {
	          this.each(function (ele) {
	            ele.style.width = value + 'px';
	          });
	          return this;
	        }
	        // get width
	        else {
	            return this[0].offsetWidth || parseFloat(this.style('width'));
	          }
	    }
	  }, {
	    key: 'height',
	    value: function height(value) {
	      var ele = this[0];
	      // window or document
	      if (ele.window === ele || ele.body) {
	        return document.body.scrollHeight > document.documentElement.scrollHeight ? document.body.scrollHeight : document.documentElement.scrollHeight;
	      }
	      // set height
	      else if (value) {
	          this.each(function (ele) {
	            ele.style.height = value + 'px';
	          });
	          return this;
	        }
	        // get height
	        else {
	            return this[0].offsetHeight || parseFloat(this.style('height'));
	          }
	    }
	  }, {
	    key: 'html',
	    value: function html(value) {
	      if (value !== undefined) {
	        this.each(function (ele) {
	          ele.innerHTML = typeof value === 'function' ? value(ele) : value;
	        });
	      } else {
	        return this[0].innerHTML;
	      }
	      return this;
	    }
	  }, {
	    key: 'text',
	    value: function text(value) {
	      if (value === undefined) return this[0].innerText || this[0].textContent;

	      this.each(function (ele) {
	        ele.innerText = ele.textContent = value;
	      });
	      return this;
	    }
	  }, {
	    key: 'val',
	    value: function val(value) {
	      if (value === undefined) return this[0].value;
	      this.each(function (ele) {
	        ele.value = value;
	      });
	      return this;
	    }
	  }, {
	    key: 'show',
	    value: function show() {
	      this.each(function (ele) {
	        ele.style.display = '';
	      });
	      return this;
	    }
	  }, {
	    key: 'hide',
	    value: function hide() {
	      this.each(function (ele) {
	        ele.style.display = 'none';
	      });
	      return this;
	    }

	    // content str || jqLite Object || DOM
	    // here is jqLite Object

	  }, {
	    key: 'append',
	    value: function append(content) {
	      this.each(function (ele) {
	        ele.appendChild(content[0]);
	      });
	      return this;
	    }
	  }, {
	    key: 'prepend',


	    // content str || jqLite Object || DOM
	    // here is jqLite Object
	    value: function prepend(content) {
	      this.each(function (ele) {
	        ele.insertBefore(content[0], ele.children[0]);
	      });
	      return this;
	    }
	  }, {
	    key: 'hasClass',
	    value: function hasClass(className) {
	      if (!this[0]) return false;
	      return this[0].classList.contains(className);
	    }
	  }, {
	    key: 'addClass',
	    value: function addClass(className) {
	      this.each(function (ele) {
	        ele.classList.add(className);
	      });
	      return this;
	    }
	  }, {
	    key: 'removeClass',
	    value: function removeClass(className) {
	      this.each(function (ele) {
	        ele.classList.remove(className);
	      });
	      return this;
	    }
	  }, {
	    key: 'index',
	    get: function get() {
	      var index = 0;
	      var brothers = this[0].parentNode.children;
	      for (var i = 0; i < brothers.length; i++) {
	        if (brothers[i] == this[0]) {
	          index = i;
	          break;
	        }
	      }
	      return index;
	    }
	  }], [{
	    key: 'fn',
	    get: function get() {
	      var visible = function visible(ele) {
	        var pos = ele.getBoundingClientRect();
	        var w = void 0;
	        var h = void 0;
	        var inViewPort = void 0;
	        var docEle = document.documentElement;
	        var docBody = document.body;
	        if (docEle.getBoundingClientRect) {
	          w = docEle.clientWidth || docBody.clientWidth;
	          h = docEle.clientHeight || docBody.clientHeight;
	          inViewPort = pos.top > h || pos.bottom < 0 || pos.left > w || pos.right < 0;
	          return inViewPort ? false : true;
	        }
	      };
	      var merge = function merge() {
	        for (var _len2 = arguments.length, sources = Array(_len2), _key2 = 0; _key2 < _len2; _key2++) {
	          sources[_key2] = arguments[_key2];
	        }

	        return Object.assign.apply(Object, [{}].concat(sources));
	      };
	      return {
	        visible: visible,
	        merge: merge
	      };
	    }
	  }]);

	  return jqLite;
	}();

	var $ = function $() {
	  var selectors = arguments.length <= 0 || arguments[0] === undefined ? '' : arguments[0];
	  var context = arguments.length <= 1 || arguments[1] === undefined ? document : arguments[1];

	  return new jqLite(selectors, context);
	};
	$.fn = jqLite.fn;

	exports.default = $;

/***/ },
/* 2 */
/***/ function(module, exports, __webpack_require__) {

	'use strict';
	var ctx         = __webpack_require__(3)
	  , $export     = __webpack_require__(5)
	  , toObject    = __webpack_require__(15)
	  , call        = __webpack_require__(17)
	  , isArrayIter = __webpack_require__(20)
	  , toLength    = __webpack_require__(24)
	  , getIterFn   = __webpack_require__(26);
	$export($export.S + $export.F * !__webpack_require__(29)(function(iter){ Array.from(iter); }), 'Array', {
	  // 22.1.2.1 Array.from(arrayLike, mapfn = undefined, thisArg = undefined)
	  from: function from(arrayLike/*, mapfn = undefined, thisArg = undefined*/){
	    var O       = toObject(arrayLike)
	      , C       = typeof this == 'function' ? this : Array
	      , $$      = arguments
	      , $$len   = $$.length
	      , mapfn   = $$len > 1 ? $$[1] : undefined
	      , mapping = mapfn !== undefined
	      , index   = 0
	      , iterFn  = getIterFn(O)
	      , length, result, step, iterator;
	    if(mapping)mapfn = ctx(mapfn, $$len > 2 ? $$[2] : undefined, 2);
	    // if object isn't iterable or it's array with default iterator - use simple case
	    if(iterFn != undefined && !(C == Array && isArrayIter(iterFn))){
	      for(iterator = iterFn.call(O), result = new C; !(step = iterator.next()).done; index++){
	        result[index] = mapping ? call(iterator, mapfn, [step.value, index], true) : step.value;
	      }
	    } else {
	      length = toLength(O.length);
	      for(result = new C(length); length > index; index++){
	        result[index] = mapping ? mapfn(O[index], index) : O[index];
	      }
	    }
	    result.length = index;
	    return result;
	  }
	});


/***/ },
/* 3 */
/***/ function(module, exports, __webpack_require__) {

	// optional / simple context binding
	var aFunction = __webpack_require__(4);
	module.exports = function(fn, that, length){
	  aFunction(fn);
	  if(that === undefined)return fn;
	  switch(length){
	    case 1: return function(a){
	      return fn.call(that, a);
	    };
	    case 2: return function(a, b){
	      return fn.call(that, a, b);
	    };
	    case 3: return function(a, b, c){
	      return fn.call(that, a, b, c);
	    };
	  }
	  return function(/* ...args */){
	    return fn.apply(that, arguments);
	  };
	};

/***/ },
/* 4 */
/***/ function(module, exports) {

	module.exports = function(it){
	  if(typeof it != 'function')throw TypeError(it + ' is not a function!');
	  return it;
	};

/***/ },
/* 5 */
/***/ function(module, exports, __webpack_require__) {

	var global    = __webpack_require__(6)
	  , core      = __webpack_require__(7)
	  , hide      = __webpack_require__(8)
	  , redefine  = __webpack_require__(13)
	  , ctx       = __webpack_require__(3)
	  , PROTOTYPE = 'prototype';

	var $export = function(type, name, source){
	  var IS_FORCED = type & $export.F
	    , IS_GLOBAL = type & $export.G
	    , IS_STATIC = type & $export.S
	    , IS_PROTO  = type & $export.P
	    , IS_BIND   = type & $export.B
	    , target    = IS_GLOBAL ? global : IS_STATIC ? global[name] || (global[name] = {}) : (global[name] || {})[PROTOTYPE]
	    , exports   = IS_GLOBAL ? core : core[name] || (core[name] = {})
	    , expProto  = exports[PROTOTYPE] || (exports[PROTOTYPE] = {})
	    , key, own, out, exp;
	  if(IS_GLOBAL)source = name;
	  for(key in source){
	    // contains in native
	    own = !IS_FORCED && target && key in target;
	    // export native or passed
	    out = (own ? target : source)[key];
	    // bind timers to global for call from export context
	    exp = IS_BIND && own ? ctx(out, global) : IS_PROTO && typeof out == 'function' ? ctx(Function.call, out) : out;
	    // extend global
	    if(target && !own)redefine(target, key, out);
	    // export
	    if(exports[key] != out)hide(exports, key, exp);
	    if(IS_PROTO && expProto[key] != out)expProto[key] = out;
	  }
	};
	global.core = core;
	// type bitmap
	$export.F = 1;  // forced
	$export.G = 2;  // global
	$export.S = 4;  // static
	$export.P = 8;  // proto
	$export.B = 16; // bind
	$export.W = 32; // wrap
	module.exports = $export;

/***/ },
/* 6 */
/***/ function(module, exports) {

	// https://github.com/zloirock/core-js/issues/86#issuecomment-115759028
	var global = module.exports = typeof window != 'undefined' && window.Math == Math
	  ? window : typeof self != 'undefined' && self.Math == Math ? self : Function('return this')();
	if(typeof __g == 'number')__g = global; // eslint-disable-line no-undef

/***/ },
/* 7 */
/***/ function(module, exports) {

	var core = module.exports = {version: '1.2.6'};
	if(typeof __e == 'number')__e = core; // eslint-disable-line no-undef

/***/ },
/* 8 */
/***/ function(module, exports, __webpack_require__) {

	var $          = __webpack_require__(9)
	  , createDesc = __webpack_require__(10);
	module.exports = __webpack_require__(11) ? function(object, key, value){
	  return $.setDesc(object, key, createDesc(1, value));
	} : function(object, key, value){
	  object[key] = value;
	  return object;
	};

/***/ },
/* 9 */
/***/ function(module, exports) {

	var $Object = Object;
	module.exports = {
	  create:     $Object.create,
	  getProto:   $Object.getPrototypeOf,
	  isEnum:     {}.propertyIsEnumerable,
	  getDesc:    $Object.getOwnPropertyDescriptor,
	  setDesc:    $Object.defineProperty,
	  setDescs:   $Object.defineProperties,
	  getKeys:    $Object.keys,
	  getNames:   $Object.getOwnPropertyNames,
	  getSymbols: $Object.getOwnPropertySymbols,
	  each:       [].forEach
	};

/***/ },
/* 10 */
/***/ function(module, exports) {

	module.exports = function(bitmap, value){
	  return {
	    enumerable  : !(bitmap & 1),
	    configurable: !(bitmap & 2),
	    writable    : !(bitmap & 4),
	    value       : value
	  };
	};

/***/ },
/* 11 */
/***/ function(module, exports, __webpack_require__) {

	// Thank's IE8 for his funny defineProperty
	module.exports = !__webpack_require__(12)(function(){
	  return Object.defineProperty({}, 'a', {get: function(){ return 7; }}).a != 7;
	});

/***/ },
/* 12 */
/***/ function(module, exports) {

	module.exports = function(exec){
	  try {
	    return !!exec();
	  } catch(e){
	    return true;
	  }
	};

/***/ },
/* 13 */
/***/ function(module, exports, __webpack_require__) {

	// add fake Function#toString
	// for correct work wrapped methods / constructors with methods like LoDash isNative
	var global    = __webpack_require__(6)
	  , hide      = __webpack_require__(8)
	  , SRC       = __webpack_require__(14)('src')
	  , TO_STRING = 'toString'
	  , $toString = Function[TO_STRING]
	  , TPL       = ('' + $toString).split(TO_STRING);

	__webpack_require__(7).inspectSource = function(it){
	  return $toString.call(it);
	};

	(module.exports = function(O, key, val, safe){
	  if(typeof val == 'function'){
	    val.hasOwnProperty(SRC) || hide(val, SRC, O[key] ? '' + O[key] : TPL.join(String(key)));
	    val.hasOwnProperty('name') || hide(val, 'name', key);
	  }
	  if(O === global){
	    O[key] = val;
	  } else {
	    if(!safe)delete O[key];
	    hide(O, key, val);
	  }
	})(Function.prototype, TO_STRING, function toString(){
	  return typeof this == 'function' && this[SRC] || $toString.call(this);
	});

/***/ },
/* 14 */
/***/ function(module, exports) {

	var id = 0
	  , px = Math.random();
	module.exports = function(key){
	  return 'Symbol('.concat(key === undefined ? '' : key, ')_', (++id + px).toString(36));
	};

/***/ },
/* 15 */
/***/ function(module, exports, __webpack_require__) {

	// 7.1.13 ToObject(argument)
	var defined = __webpack_require__(16);
	module.exports = function(it){
	  return Object(defined(it));
	};

/***/ },
/* 16 */
/***/ function(module, exports) {

	// 7.2.1 RequireObjectCoercible(argument)
	module.exports = function(it){
	  if(it == undefined)throw TypeError("Can't call method on  " + it);
	  return it;
	};

/***/ },
/* 17 */
/***/ function(module, exports, __webpack_require__) {

	// call something on iterator step with safe closing on error
	var anObject = __webpack_require__(18);
	module.exports = function(iterator, fn, value, entries){
	  try {
	    return entries ? fn(anObject(value)[0], value[1]) : fn(value);
	  // 7.4.6 IteratorClose(iterator, completion)
	  } catch(e){
	    var ret = iterator['return'];
	    if(ret !== undefined)anObject(ret.call(iterator));
	    throw e;
	  }
	};

/***/ },
/* 18 */
/***/ function(module, exports, __webpack_require__) {

	var isObject = __webpack_require__(19);
	module.exports = function(it){
	  if(!isObject(it))throw TypeError(it + ' is not an object!');
	  return it;
	};

/***/ },
/* 19 */
/***/ function(module, exports) {

	module.exports = function(it){
	  return typeof it === 'object' ? it !== null : typeof it === 'function';
	};

/***/ },
/* 20 */
/***/ function(module, exports, __webpack_require__) {

	// check on default Array iterator
	var Iterators  = __webpack_require__(21)
	  , ITERATOR   = __webpack_require__(22)('iterator')
	  , ArrayProto = Array.prototype;

	module.exports = function(it){
	  return it !== undefined && (Iterators.Array === it || ArrayProto[ITERATOR] === it);
	};

/***/ },
/* 21 */
/***/ function(module, exports) {

	module.exports = {};

/***/ },
/* 22 */
/***/ function(module, exports, __webpack_require__) {

	var store  = __webpack_require__(23)('wks')
	  , uid    = __webpack_require__(14)
	  , Symbol = __webpack_require__(6).Symbol;
	module.exports = function(name){
	  return store[name] || (store[name] =
	    Symbol && Symbol[name] || (Symbol || uid)('Symbol.' + name));
	};

/***/ },
/* 23 */
/***/ function(module, exports, __webpack_require__) {

	var global = __webpack_require__(6)
	  , SHARED = '__core-js_shared__'
	  , store  = global[SHARED] || (global[SHARED] = {});
	module.exports = function(key){
	  return store[key] || (store[key] = {});
	};

/***/ },
/* 24 */
/***/ function(module, exports, __webpack_require__) {

	// 7.1.15 ToLength
	var toInteger = __webpack_require__(25)
	  , min       = Math.min;
	module.exports = function(it){
	  return it > 0 ? min(toInteger(it), 0x1fffffffffffff) : 0; // pow(2, 53) - 1 == 9007199254740991
	};

/***/ },
/* 25 */
/***/ function(module, exports) {

	// 7.1.4 ToInteger
	var ceil  = Math.ceil
	  , floor = Math.floor;
	module.exports = function(it){
	  return isNaN(it = +it) ? 0 : (it > 0 ? floor : ceil)(it);
	};

/***/ },
/* 26 */
/***/ function(module, exports, __webpack_require__) {

	var classof   = __webpack_require__(27)
	  , ITERATOR  = __webpack_require__(22)('iterator')
	  , Iterators = __webpack_require__(21);
	module.exports = __webpack_require__(7).getIteratorMethod = function(it){
	  if(it != undefined)return it[ITERATOR]
	    || it['@@iterator']
	    || Iterators[classof(it)];
	};

/***/ },
/* 27 */
/***/ function(module, exports, __webpack_require__) {

	// getting tag from 19.1.3.6 Object.prototype.toString()
	var cof = __webpack_require__(28)
	  , TAG = __webpack_require__(22)('toStringTag')
	  // ES3 wrong here
	  , ARG = cof(function(){ return arguments; }()) == 'Arguments';

	module.exports = function(it){
	  var O, T, B;
	  return it === undefined ? 'Undefined' : it === null ? 'Null'
	    // @@toStringTag case
	    : typeof (T = (O = Object(it))[TAG]) == 'string' ? T
	    // builtinTag case
	    : ARG ? cof(O)
	    // ES3 arguments fallback
	    : (B = cof(O)) == 'Object' && typeof O.callee == 'function' ? 'Arguments' : B;
	};

/***/ },
/* 28 */
/***/ function(module, exports) {

	var toString = {}.toString;

	module.exports = function(it){
	  return toString.call(it).slice(8, -1);
	};

/***/ },
/* 29 */
/***/ function(module, exports, __webpack_require__) {

	var ITERATOR     = __webpack_require__(22)('iterator')
	  , SAFE_CLOSING = false;

	try {
	  var riter = [7][ITERATOR]();
	  riter['return'] = function(){ SAFE_CLOSING = true; };
	  Array.from(riter, function(){ throw 2; });
	} catch(e){ /* empty */ }

	module.exports = function(exec, skipClosing){
	  if(!skipClosing && !SAFE_CLOSING)return false;
	  var safe = false;
	  try {
	    var arr  = [7]
	      , iter = arr[ITERATOR]();
	    iter.next = function(){ safe = true; };
	    arr[ITERATOR] = function(){ return iter; };
	    exec(arr);
	  } catch(e){ /* empty */ }
	  return safe;
	};

/***/ },
/* 30 */
/***/ function(module, exports, __webpack_require__) {

	// 19.1.3.1 Object.assign(target, source)
	var $export = __webpack_require__(5);

	$export($export.S + $export.F, 'Object', {assign: __webpack_require__(31)});

/***/ },
/* 31 */
/***/ function(module, exports, __webpack_require__) {

	// 19.1.2.1 Object.assign(target, source, ...)
	var $        = __webpack_require__(9)
	  , toObject = __webpack_require__(15)
	  , IObject  = __webpack_require__(32);

	// should work with symbols and should have deterministic property order (V8 bug)
	module.exports = __webpack_require__(12)(function(){
	  var a = Object.assign
	    , A = {}
	    , B = {}
	    , S = Symbol()
	    , K = 'abcdefghijklmnopqrst';
	  A[S] = 7;
	  K.split('').forEach(function(k){ B[k] = k; });
	  return a({}, A)[S] != 7 || Object.keys(a({}, B)).join('') != K;
	}) ? function assign(target, source){ // eslint-disable-line no-unused-vars
	  var T     = toObject(target)
	    , $$    = arguments
	    , $$len = $$.length
	    , index = 1
	    , getKeys    = $.getKeys
	    , getSymbols = $.getSymbols
	    , isEnum     = $.isEnum;
	  while($$len > index){
	    var S      = IObject($$[index++])
	      , keys   = getSymbols ? getKeys(S).concat(getSymbols(S)) : getKeys(S)
	      , length = keys.length
	      , j      = 0
	      , key;
	    while(length > j)if(isEnum.call(S, key = keys[j++]))T[key] = S[key];
	  }
	  return T;
	} : Object.assign;

/***/ },
/* 32 */
/***/ function(module, exports, __webpack_require__) {

	// fallback for non-array-like ES3 and non-enumerable old V8 strings
	var cof = __webpack_require__(28);
	module.exports = Object('z').propertyIsEnumerable(0) ? Object : function(it){
	  return cof(it) == 'String' ? it.split('') : Object(it);
	};

/***/ },
/* 33 */,
/* 34 */,
/* 35 */,
/* 36 */,
/* 37 */,
/* 38 */,
/* 39 */,
/* 40 */,
/* 41 */,
/* 42 */,
/* 43 */,
/* 44 */,
/* 45 */,
/* 46 */,
/* 47 */,
/* 48 */,
/* 49 */,
/* 50 */,
/* 51 */,
/* 52 */,
/* 53 */,
/* 54 */,
/* 55 */
/***/ function(module, exports) {

	"use strict";

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	var $interval = function $interval(fn, delay) {
	  var interval = function interval() {
	    fn.call(undefined);
	    id = setTimeout(interval, delay);
	  };

	  var id = setTimeout(interval, delay);

	  return function () {
	    window.clearTimeout(id);
	  };
	};

	$interval.cancel = function (timerFunc) {
	  timerFunc();
	};

	exports.default = $interval;

/***/ },
/* 56 */
/***/ function(module, exports, __webpack_require__) {

	'use strict';

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});

	var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();

	var _jqLite = __webpack_require__(1);

	var _jqLite2 = _interopRequireDefault(_jqLite);

	var _config = __webpack_require__(57);

	var _config2 = _interopRequireDefault(_config);

	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

	function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

	var Main = function () {
	  function Main() {
	    var rules = arguments.length <= 0 || arguments[0] === undefined ? _config2.default.rules : arguments[0];

	    _classCallCheck(this, Main);

	    this.ads = (0, _jqLite2.default)(rules);
	    this.length = this.ads.length;
	  }

	  _createClass(Main, [{
	    key: 'filter',
	    value: function filter() {
	      this.ads.each(function (ele, i) {
	        ele.style.cssText = '\n          display:none !important;\n          visibility:hidden !important;\n          width:0 !important;\n          height:0 !important;\n          overflow:hidden !important;\n          // background-color:red !important;\n          // border:1px solid red;\n        ';
	        ele.setAttribute('filtered', '');
	      });
	      return this;
	    }
	  }, {
	    key: 'turn',
	    value: function turn() {
	      (0, _jqLite2.default)('#content_left input[type=checkbox]:not(filtered)').each(function (ele) {
	        ele.checked = false;
	        ele.setAttribute('filtered', '');
	      });
	      return this;
	    }
	  }]);

	  return Main;
	}();

	exports.default = Main;

/***/ },
/* 57 */
/***/ function(module, exports) {

	'use strict';

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	var CONFIG = {
	  rules: '\n    #content_left>div\n    :not([class*=result])\n    :not([class*=container])\n    :not(.leftBlock)\n    :not(#rs_top_new)\n    :not([filtered])\n    ,\n    #content_left>table\n    :not(.result)\n    :not([filtered])\n    ,\n    #content_right>table td\n    div#ec_im_container\n    ,\n    div.s-news-list-wrapper>div\n    :not([data-relatewords*="1"])\n    ,\n    div.list-wraper dl[data-oad]\n    :not([data-fb])\n    :not([filtered])\n  '.trim().replace(/\n/img, '').replace(/\s{1,}([^a-zA-Z])/g, '$1')
	};

	exports.default = CONFIG;

/***/ }
/******/ ]);


// ==UserScript==
// @name         网盘提取工具
// @namespace    http://www.fishlee.net/
// @version      2.7
// @description  尽可能在支持的网盘自动输入提取码，省去下载的烦恼。
// @author       木鱼(iFish)
// @match        *://*/*
// @grant        unsafeWindow
// @icon         https://ssl-static.fishlee.net/resources/emot/xr/22.gif
// ==/UserScript==
(function(window, self, unsafeWindow) {
    'use strict';
    var timeStart = new Date().getTime();
    var location = self.location;
    var host = location.host;
    var path = location.pathname;
    var code, input;
    var getCode = function(rule) {
        code = location.hash.slice(1, 5);
        if ((rule || /([a-z\d]{4})/i.exec(code))) {
            code = RegExp.$1;
        } else code = null;
        return code;
    };
    if (/(pan|e?yun)\.baidu\.com/.test(host)) {
        //百度云盘
        if (path.indexOf("/share/") !== -1 && document.querySelector('form[name="accessForm"]') && getCode()) {
            var target=document.querySelector('.pickpw input');
            if(!target)
                return;

            target.value = code;
            unsafeWindow.document.querySelector('form[name="accessForm"]').onsubmit();
        }
    } else {
        //其它网站，检测链接
        Array.prototype.slice.call(document.querySelectorAll("a[href*='pan.baidu.com'], a[href*='yunpan.cn'], a[href*='vdisk.weibo.com']")).forEach(function(link) {
            var txt = link.nextSibling && link.nextSibling.nodeValue;
            var linkcode = /码.*?([a-z\d]{4})/i.exec(txt) && RegExp.$1;
            if (!linkcode) {
                txt = link.parentNode.innerText;
                linkcode = /码.*?([a-z\d]{4})/i.exec(txt) && RegExp.$1;
            }
            if (linkcode) {
                var href = link.getAttribute("href");
                link.setAttribute("href", href + "#" + linkcode);
            }
        });
    }
    var timeEnd = new Date().getTime();
    console.log("[网盘提取工具] 链接处理完成，耗时：" + (timeEnd - timeStart) + "毫秒. 处理模式：DOM处理");
})(window, window.self, unsafeWindow);
(function() {
    'use strict';
    //consts...
    var CODE_RULE_BAIDU = /^([a-z\d]{4})$/i;
    var CODE_RULE_YUNPAN = /^([a-z\d]{4})$/i;
    var MAX_SEARCH_CODE_RANGE = 5;
    //functions...
    var textNodesUnder = function(el) {
        var n, a = [],
            walk = document.createTreeWalker(el, NodeFilter.SHOW_TEXT, null, false);
        while ((n = walk.nextNode())) {
            if (n.nodeName === '#text')
                a.push(n);
        }
        return a;
    };
    var generalLinkifyText = function(text, eles, index, testReg, validateRule) {
        var loopCount = 0,
            originalText, code, match, url,
            linkifiedText = text;
        while ((match = testReg.exec(text))) {
            loopCount++;
            url = (match[1] || "http://") + match[2];
            originalText = (match[1] || "") + match[2];
            code = match[3] || findCodeFromElements(eles, index, validateRule) || "";
            if (!code)
                continue;
            console.log("[网盘提取工具] 已处理网盘地址，URL=" + url + "，提取码=" + code + "模式：TEXTNODE");
            //fix double #
            url = url.split('#')[0];
            linkifiedText = linkifiedText.replace(originalText, "<a href='" + url + "#" + code + "' target='_blank'>" + url + '</a>');
        }
        return [loopCount, linkifiedText];
    };
    var linkifyTextBlockBaidu = function(text, eles, index) {
        return generalLinkifyText(text, eles, index, /(https?:\/\/)?((?:pan|e?yun)\.baidu\.com\/s\/(?:[a-z\d]+)(?:#[a-z\d-_]*)?)(?:.*?码.*?([a-z\d]+))?/gi, CODE_RULE_BAIDU);
    };
    //var linkifyTextBlockYunpan = function(text, eles, index) {
    //    return generalLinkifyText(text, eles, index, /(https?:\/\/)?(yunpan\.cn\/(?:[a-z\d]+))(?:.*?码.*?([a-z\d]+))?/gi, CODE_RULE_YUNPAN);
    //};
    var findCodeFromElements = function(eles, index, rule) {
        for (var i = 0; i < MAX_SEARCH_CODE_RANGE && i < eles.length; i++) {
            var txt = eles[i + index].textContent;
            var codeReg = /码.*?([a-z\d]+)/gi;
            var codeMatch = codeReg.exec(txt) && RegExp.$1;
            if (!codeMatch) continue;
            var linkTestReg = /(https?:|\.(net|cn|com|gov|cc|me))/gi;
            if (linkTestReg.exec(txt) && linkTestReg.lastIndex <= codeReg.lastIndex) {
                break;
            }
            if (rule.test(codeMatch)) return codeMatch;
        }
        return null;
    };
    var linkify = function() {
        var eles = textNodesUnder(document.body);
        var ele, txt, loopCount;
        var processor = [
            linkifyTextBlockBaidu
        ];
        var callback = function(fun) {
            var data = fun(txt, eles, i + 1);
            loopCount += data[0];
            txt = data[1];
        };
        for (var i = 0; i < eles.length; i++) {
            ele = eles[i];
            if (ele.parentNode.tagName == 'a') continue;
            txt = ele.textContent;
            loopCount = 0;
            processor.forEach(callback);
            if (loopCount > 0) {
                var span = document.createElement("span");
                span.innerHTML = txt;
                ele.parentNode.replaceChild(span, ele);
            }
        }
    };
    var timeStart = new Date().getTime();
    linkify();
    var timeEnd = new Date().getTime();
    console.log("[网盘提取工具] 链接处理完成，耗时：" + (timeEnd - timeStart) + "毫秒. 处理模式：TEXTNODE处理");
})();


// ==UserScript==
// @name 			网盘自动填写密码【增强版】
// @description		网盘自动填写提取密码【增强版】+网盘超链接与提取码融合。
// @author			极品小猫
// @namespace   	http://www.cnblogs.com/hkmhd/
// @homepage		https://greasyfork.org/scripts/13463
// @supportURL		https://greasyfork.org/scripts/13463/feedback
// @version			2.5.3
// @date			2015.10.30
// @modified		2017.07.19
// 
// 支持的网盘
// @include     	http://*
// @include			https://*
// @include			http://pan.baidu.com/s/*
// @include			http://eyun.baidu.com/s/*
// 
// 白名单
// @exclude			http*://*.pcs.baidu.com/*
// @exclude			http*://*.baidupcs.com/*
// @exclude			http*://*:8666/file/*
// @exclude			http*://*.baidu.com/file/*
// @exclude			http*://index.baidu.com/*
// 
// @exclude			http*://*.gov/*
// @exclude			http*://*.gov.cn/*
// @exclude			http*://*.taobao.com/*
// @exclude			http*://*.tmall.com/*
// @exclude			http*://*.alimama.com/*
// @exclude			http*://*.jd.com/*
// @exclude			http://*.ctrip.com/*
// @exclude			https://*.evernote.com/*
// @exclude			https://*.yinxiang.com/*
// @exclude			/^https?://(localhost|10\.|192\.|127\.)/
// @exclude			/https?://www.baidu.com/(?:s|baidu)\?/
// @exclude			http*://www.zhihu.com/question/*/answers/created
//
// require			http://code.jquery.com/jquery-2.1.4.min.js
// @require			http://cdn.staticfile.org/jquery/2.1.4/jquery.min.js
// @grant			unsafeWindow
// @encoding		utf-8
// @run-at			document-idle
// ==/UserScript==

var urls=location.href;
var hash=location.hash;
var host=location.hostname.replace(/^www\./i,'').toLowerCase();
var paths=location.pathname.toLowerCase();
unsafeWindow.eve = Event;

/*
 * 更新日志
 * 2.5.3 [2017-11-08]
 * 1、【修正】百度网盘无法自动填写密码问题（估计此次更新为反脚本和外挂程序的策略，此为临时修正，其它功能是否有影响待测试）
 
 * 2.5.2.3
 * 1、MAD素材库（madsck.com）密码融合特殊支持
 * 2、修复 huhupan.com 的密码融合支持
 *
 * 2.5.2.2
 * 1、删除知乎跳转链处理代码
 *
 * 2.5.2.1
 * 1、讯影网（xunyingwang.com）密码融合处理
 * 2、户户盘（huhupan.com）密码融合处理
 * 
 * 2.5.1
 * 1、殁漂遥（shaoit.com）密码融合处理改进
 * 
 * 2.5.0
 * 1、提升密码融合能力
 * 
 * 2.4.2 【2016.10.05】
 * 1、殁漂遥（shaoit.com）密码融合处理
 * 2、reimu.net 密码融合处理
 * 3、修复“跳转链处理”影响百度企业盘无法访问问题
 * 
 * 2.4.1.2 【2016.09.28】
 * 1、增加 sijihuisuo.club 的跳转链处理
 * 2、增加本地IP白名单
 * 
 * 2.4.1.1 【2016.09.21】
 * 1、增加携程网白名单
 * 
 * 2.4.1 【2016.09.21】
 * 1、支持一些特殊网站的密码融合预处理
 * 2、支持百度企业云的密码自动提交（企业盘的密码为4~14位）
 * 3、支持 www.0dayku.com 的提取码关键字“Extracted-code”
 * 
 * 2.3.3 【2016.08.08】
 * 1、知乎跳转链预处理
 * 
 * 2.3.2 【2016.08.04】
 * 1、恢复提取码中的“密码”关键字（适用于：心海e站）
 * 
 * 2.3.1 【2016.08.03】
 * 1、增加微云网盘提取码支持（匹配规则来自原作者 Jixun.Moe）
 * 2、修正提取码兼容问题
 * 3、修正重复添加提取码
 * 
 * 2.3.0 【2016.08.01】
 * 1、移除对金山快盘、新浪云盘的支持
 * 2、百度企业云盘不追加验证码
 * 3、受贴吧页面跳转影响，暂时不支持贴吧的密码提取，已将贴吧加入白名单
 * 4、提升链接&密码融合的成功率	—— A 标签绑定函数更改为 body 点击事件监听（根据原作者 Jixun.Moe 的建议）
 * 5、找不到密码时的遍历方式更改（感谢 10139 - mudoo 的建议）
 * 6、支持密码放在换行表格中的提取
*/

var site = {
  'baidu.com':{
    chk:	/^[a-z0-9]{4}$/,
    code:	'.pickpw input, #accessCode',
    btn:	'.g-button, #submitBtn, #getfileBtn',
    PreProcess: function() {	//已处理
      if(host=='eyun.baidu.com'){	//如果为百度企业云盘，则按照企业云盘的方式处理
        conf=site['baidu.com']=site['eyun.baidu.com'];
        site['baidu.com'].PreProcess();
      }
    }
  },
  'eyun.baidu.com': {
    chk:	/^[a-z0-9]{4}$/,
    code:	'.share-access-code',
    btn:	'.g-button-right',
    PreProcess: function() {
      if((hash&&!/sharelink|path/i.test(hash))&&!/enterprise/.test(paths)) {
        console.log('test');
        location.href=location.href.replace(location.hash,'');
      }
    }
  },
  'weiyun.com': {
    chk: /^[a-z0-9]{4}$/i,
    code: '#outlink_pwd',
    btn:  '#outlink_pwd_ok'
  },
  'pwdRule' : /(?:提取|访问)[码碼]?\s*[:： ]?\s*([a-z\d]{4})/,			//常规密码
  'codeRule' : /(?:(?:提取|访问|密[码碼]|艾|Extracted-code)[码碼]?)\s*[:： ]?\s*([a-z\d]{4})/i,	//其它类型的密码
  //跳转链预处理
  'JumpUrl' : {
    'sijihuisuo.club': {
      href: $('.down-tip A[href^="https://www.sijihuisuo.club/go/?url="]'),
      url: 'https://www.sijihuisuo.club/go/?url='
    }
  },
  //密码融合需要特殊支持的网站
  'Support' : {
    'madsck.com':{
      path: /\/resource\/\d+/,
      callback:function(){
        var ID=$('.btn-download').data('id');
          $.ajax({
            "url":"http://www.madsck.com/ajax/login/download-link?id="+ID,
            method: "GET",
            dataType: "json",
            success:function(e){
              var res=e.resource;
              $('.btn-download').css('display','none');
              $('<a>').attr({'href':res.resource_link+'#'+res.fetch_code,'target':'blank','class':'btn-download'}).css({'line-height':'60px','text-align':'center','font-size':'24px'}).text('下载').insertBefore('.btn-download');
              console.log(e);
            }
          })
      }
    },
    'idanmu.co': {
      path : /storage\-download/i,
      callback : function(){
        $('.input-group').each(function(){
          $(this).text($(this).text()+$(this).find('input').val());
        });
      }
    },
    'shaoit.com': {
      path : /.*/i,
      callback : function(){
        var LinkParent=$('A[href*="pan.baidu.com"],A[href*="eyun.baidu.com"]').parent();
        var ParentHTML=LinkParent.html();
        site['codeRule']=/\s*(shaoit|[a-z\d]{4})((?:\s*|<br>)$)/i;
          var HtmlArr=ParentHTML.match(/.*(?: \/|\s*$|<br>)/ig);
          for(i=0;i<HtmlArr.length;i++){
            if(/<\/a>\s*([a-z\d]{4}|shaoit) \//i.test(HtmlArr[i])){
              HtmlArr[i]=HtmlArr[i].replace(/(href="[^"]+)("[^>]*?>(?!<\/a>).+?<\/a>\s*(shaoit|[a-z\d]{4}))/ig,'$1#$3$2');
            } else if(site['codeRule'].test(HtmlArr[i])){
              var PWcode=HtmlArr[i].match(site['codeRule'])[1];
              HtmlArr[i]=HtmlArr[i].replace(/(href="[^"]+?)"/ig,'$1#'+PWcode+'"');
            }
          }
          LinkParent.html(HtmlArr.join(''));
        
      }
    },
    'xunyingwang.com':{
      path:/movie/i,
      callback:function(){
        $(window).load(function(){
          $('A[href*="pan.baidu.com"],A[href*="eyun.baidu.com"]').each(function(){
            $(this).attr('href',$(this).attr('href')+'#'+$(this).next("strong").text())
          })
        })
      }
    },
    'huhupan.com':{
      path:/e\/extend\/down/i,
      callback:function(){
        var _Linktmp=$('A[href*="pan.baidu.com"],A[href*="eyun.baidu.com"]');
        var _PWtmp=$('input[id^="bdypas"]');
        for(i=0;i<_Linktmp.length;i++){
          _Linktmp[i].href+="#"+_PWtmp[i].value;
        }
      }
    },
    'reimu.net': {
      path: /archives/i,
      callback: function(){
        site['codeRule']=/(?:(?:提取|访问|密[码碼])[码碼]?)\s*[:： ]?\s*([a-z\d]{4}|8酱)/i
      }
    }
  }
};

var hostName = location.host.match(/\w+\.\w+$/)[0].toLowerCase();	//提取当前网站主域名（网盘填充密码用）
var conf = site[hostName];											//设置主域名

var HostArr = [];									//生成域名数组
for(var i in site) HostArr.push(i);					//插入域名对象数组
var HostExp = new RegExp(HostArr.join("|"),'i');	//生成校验超链接的正则

//console.log(site.JumpUrl[host]);

/* -----===== 检查是否需要处理跳转链 Start =====----- */

if(site['JumpUrl'][host]){
  console.log(site['JumpUrl'][host]['href'])
  site['JumpUrl'][host]['href'].each(function(){
    console.log(site['JumpUrl'][host]['rep'])
    $(this).attr({'href':decodeURIComponent($(this).attr('href').replace(site['JumpUrl'][host]['url'],'')),'target':'blank'});
  });
}
/* -----===== 检查是否需要处理跳转链 End =====----- */


if(conf&&!/zhidao.baidu.com/i.test(host)){	//网盘页面填密码登录
  // 抓取提取码
  if(conf.PreProcess) conf.PreProcess();		//内容预处理（处理百度企业云）
  var sCode = hash.slice(1).trim();

  // 调试用，检查是否为合法格式
  if (!conf.chk.test(sCode)) {
    console.log('没有 Key 或格式不对');
  } else {
    console.log ('抓取到的提取码: %s', sCode);
  }

  // 加个小延时
  setTimeout (function () {
    // 键入提取码并单击「提交」按钮，报错不用理。
    var codeBox = $(conf.code),
        btnOk = $(conf.btn);
    if(codeBox.length>0) {		//存在密码框时才进行密码提交操作
      codeBox.val(sCode);		//填写验证码
      if (conf.preSubmit)
        if (conf.preSubmit (codeBox, btnOk))
          return ;
      btnOk.click();
    }
  }, 10);
} else {
  //密码融合 特别支持的网站
  if(site['Support'][hostName]&&site['Support'][hostName]['path'].test(paths)) {
    site['Support'][hostName].callback();
  }
  //监听 A 标签点击事件
  $('body').on('click', 'a', function () {
    var target=this;
    //如果超链接已有 hash 则跳过
    if(this.hash) return;
    //如果目标对象为百度企业盘，提升密码匹配范围，以兼容百度企业云
    if(/eyun.baidu.com/i.test(this.href)) {
  		site['pwdRule']=/(?:提取|访问)[码碼]?\s*[:： ]?\s*([a-z\d]{4,14})/;
  		site['codeRule']=/(?:(?:提取|访问|密[码碼]|Extracted-code)[码碼]?)\s*[:： ]?\s*([a-z\d]{4,14})/i;
    }
    //var target=event.target.tagName==='A'?event.target:event.target.parentNode;
   	
    //正则校验超链接匹配的网盘
    if(HostExp.test(this.href)&&!/(?:tieba)\.baidu\.com/i.test(this.href)){
      
      
      if(site['codeRule'].test(target.textContent)){
        console.log('在当前超链接的对象中查找密码');
        target.href+='#'+extCode(target);
      } else if(target.nextSibling&&site['codeRule'].test(target.nextSibling.textContent)){
        console.log('密码在超链接后面的兄弟元素中',target.nextSibling.textContent);
        if(!/#/i.test(target.href)) target.href+='#'+extCode(target.nextSibling);
      } else if(site['pwdRule'].test(target.parentNode.textContent)){
        console.log('从父对象中查找密码');
        if(!/#/i.test(target.href)) target.href+='#'+extCode(target.parentNode);
      } else {
        var i = 0,
            maxParent = 5,	//向上遍历的层级
            parent = target;
        while(i<maxParent) {
          i++;									//遍历计数
          parent = parent.parentNode;			//取得父对象
          console.log('遍历上级目录查找密码：'+ i,parent);
          if(parent.tagName=="TR") {				//如果父对象是表格，则从表格中提取密码
            if(site['codeRule'].test(parent.nextElementSibling.textContent)) {
              parent=parent.nextElementSibling;
              //console.log('表格中查找密码成功！',parent);
              target.href+='#'+extCode(parent);
              break;
            }
          } else if(site['codeRule'].test(pw=parent.nextSibling.textContent)){
            console.log('向上遍历查找，在超链接后面的兄弟元素中，',parent.nextSibling);
            target.href+='#'+extCode(parent.nextSibling);
            break;
          } else if(site['codeRule'].test(parent.textContent)) {		//否则按照常规方式提取密码
            console.log('向上遍历查找密码成功！');
            target.href+='#'+extCode(parent);
            break;
          }
          if(parent==document.body) break;								//如果已经遍历到最顶部
        }
      }
      //console.log(site['codeRule']);
            //旧的从父对象中遍历方式
            //console.log('从 document.body 中查找密码');
            //if(!/#/i.test(target.href)) target.href+='#'+extCode(document.body);
    }
  });
}

function extCode(obj){
  text=obj.textContent.trim();
  return site['pwdRule'].test(text)?text.match(site['pwdRule'])[1]:text.match(site['codeRule'])[1];	//首先尝试使用 提取码|访问码 作为密码匹配的关键字，无效时则使用更完整的匹配规则
}


// ==UserScript==
// @name        阻止百度网盘wap版自动跳转
// @namespace   https://greasyfork.org/users/4514
// @author      喵拉布丁
// @homepage    https://greasyfork.org/scripts/13434
// @description 阻止百度网盘wap版自动跳转到PC版网页（仅限Firefox浏览器有效）
// @include     http://pan.baidu.com/wap/*
// @include     https://pan.baidu.com/wap/*
// @include     http://yun.baidu.com/wap/*
// @include     https://yun.baidu.com/wap/*
// @version     1.3
// @grant       none
// @run-at      document-start
// @license     MIT
// ==/UserScript==
document.addEventListener('beforescriptexecute', function (e) {
    if (e.target.id == 'platform') {
        e.preventDefault();
    }
});


// ==UserScript==
// @name            Fuck百度云
// @description     就他妈不装百度云管家
// @namespace       http://www.jycggyh.cn/
// @author          艮古永恒
// @version         1.1.0
// @include         pan.baidu.com/*
// @match           *://pan.baidu.com/*
// @grant           none
// @run-at          document-end
// @require         https://greasyfork.org/scripts/21104-%E6%88%91%E7%9A%84js%E5%87%BD%E6%95%B0%E5%BA%93/code/%E6%88%91%E7%9A%84JS%E5%87%BD%E6%95%B0%E5%BA%93.user.js
// ==/UserScript==

function addToolbarBtn2(name, iconClass) {
  // toolbar Div
  var oToolbar = queryByCName("bar").getElementsByTagName("div")[1];
  // new btn
  var oButton = document.createElement("a");
  oButton.className = "g-button";
  oButton.href="javascript:void(0)";
  oToolbar.appendChild(oButton);
  // btn -> span
  var oBtnRoot = document.createElement("span");
  oBtnRoot.className = "g-button-right";
  oButton.appendChild(oBtnRoot);
  // btn -> span -> em
  oBtnIcon = document.createElement("em");
  oBtnIcon.className = iconClass;
  oBtnIcon.title = name;
  oBtnRoot.appendChild(oBtnIcon);
  // btn -> span -> text
  oBtnText = document.createElement("span");
  oBtnText.className = "text";
  oBtnText.innerHTML = name;
  oBtnRoot.appendChild(oBtnText);
  // set up end
  return oButton;
}

function getCurrentPath2() {
  var oUl = queryByCName("historylistmanager-history");
  var oLi = queryByTName("li", oUl)[1];
  var oSpans = oLi.getElementsByTagName("span");
  if(oSpans.length == 0) {
    return "";
  }
  var oSpan = oSpans[oSpans.length-1];
  return oSpan.title.substring(4);
}

// 初始化设置
var DownloadAPI = "https://pcs.baidu.com/rest/2.0/pcs/file?method=download&app_id=266719&path=";
var oBtnDown = addToolbarBtn2("默认操作", "icon icon-download-gray");
var DOWNLOAD = false;
var oBtnIcon;
var oBtnText;
oBtnDown.onclick = function () {
  DOWNLOAD = !DOWNLOAD;
  updateBtn();
  
  if(!DOWNLOAD) {
    window.location.reload(true);
  }
  updateFiles();
}

function updateFiles() {
  if(!DOWNLOAD) {
  	return; 
  }
  var oFiles = queryByCName("list-view").getElementsByTagName("dd");
  var CurrentPath = getCurrentPath2();
  var oFile;
  for(var i = 0; i < oFiles.length; i++) {
      oFile = oFiles[i].getElementsByClassName("file-name")[0].getElementsByClassName("text")[0].getElementsByClassName("filename")[0];
      oFile.onclick = function() {
        window.location.href = DownloadAPI + CurrentPath + "/" + this.title;
      }
  }
}

function updateBtn() {
  if(!DOWNLOAD) {
  	oBtnIcon.title = "默认操作";
    oBtnText.innerHTML = "默认操作";
  } else {
    oBtnIcon.title = "下载操作";
    oBtnText.innerHTML = "下载操作";
  }
}


// ==UserScript==
// @name        百度网盘助手•改
// @author      有一份田
// @description 显示百度网盘文件的直接链接,突破大文件需要使用电脑管家的限制
// @namespace   https://greasyfork.org/zh-CN/scripts/986-百度网盘助手
// @icon        http://img.duoluohua.com/appimg/script_dupanlink_icon_48.png
// @license     GPL version 3
// @encoding    utf-8
// @date        26/08/2013
// @modified    07/18/2016
// @include     https://pan.baidu.com/*
// @include     http://pan.baidu.com/*
// @include     http://yun.baidu.com/*
// @exclude     http://yun.baidu.com
// @exclude     http://yun.baidu.com/#*
// @exclude     http://pan.baidu.com/share/manage*
// @exclude     http://pan.baidu.com/disk/recyclebin*
// @exclude     http://yun.baidu.com/pcloud/album/info*
// @require     http://code.jquery.com/jquery-2.1.1.min.js
// @grant       unsafeWindow
// @grant       GM_setClipboard
// @run-at      document-end
// @version     3.1.0
// ==/UserScript==


/*
 * === 说明 ===
 *@作者:有一份田
 *@官网:http://www.duoluohua.com/download/
 *@Email:youyifentian@gmail.com
 *@Git:http://git.oschina.net/youyifentian
 *@转载重用请保留此信息
 *
 *
 * */





var VERSION = '3.0.2';
var APPNAME = '\u767e\u5ea6\u7f51\u76d8\u52a9\u624b';
var t = new Date().getTime();


$ = $ || unsafeWindow.$;
var disk = unsafeWindow.disk;
var FileUtils = unsafeWindow.FileUtils;
var Page = unsafeWindow.Page;
var Utilities = unsafeWindow.Utilities;
var yunData = unsafeWindow.yunData;
var require= unsafeWindow.require;


(function (){
    var isOther = location.href.indexOf('://pan.baidu.com/disk')==-1,
    downProxy = null,shareData = null,
    Canvas,Pancel,RestAPI,Toast={},errorMsg,
    iframe = '',httpHwnd = null,index = 0,
    msg = [
        '\u54b1\u80fd\u4e0d\u4e8c\u4e48,\u4e00\u4e2a\u6587\u4ef6\u90fd\u4e0d\u9009\u4f60\u8ba9\u6211\u548b\u4e2a\u529e...', //0
        '\u5c3c\u739b\u4e00\u4e2a\u6587\u4ef6\u90fd\u4e0d\u9009\u4f60\u4e0b\u4e2a\u6bdb\u7ebf\u554a...', //1
        '\u4f60TM\u77e5\u9053\u4f60\u9009\u4e86<b>90</b>\u591a\u4e2a\u6587\u4ef6\u5417?\u60f3\u7d2f\u6b7b\u6211\u554a...', //2
        '<b>\u8bf7\u6c42\u5df2\u53d1\u9001\uff0c\u6570\u636e\u4e0b\u884c\u4e2d...</b>', //3
        '<b>\u8be5\u9875\u9762</b>\u4e0d\u652f\u6301\u6587\u4ef6\u5939\u548c\u591a\u6587\u4ef6\u7684<font color="red"><b>\u94fe\u63a5\u590d\u5236\u548c\u67e5\u770b</b></font>\uff01', //4
        '<font color="red">\u8bf7\u6c42\u8d85\u65f6\u4e86...</font>', //5
        '<font color="red">\u8bf7\u6c42\u51fa\u9519\u4e86...</font>', //6
        '<font color="red">\u8fd4\u56de\u6570\u636e\u65e0\u6cd5\u76f4\u89c6...</font>', //7
        '\u8bf7\u8f93\u5165\u9a8c\u8bc1\u7801', //8
        '\u9a8c\u8bc1\u7801\u8f93\u5165\u9519\u8bef,\u8bf7\u91cd\u65b0\u8f93\u5165', //9
        '<b>\u94fe\u63a5\u5df2\u590d\u5236\u5230\u526a\u5207\u677f\uff01</b>', //10
        '\u672a\u77e5\u9519\u8bef\uff0cerrno:',//11
        '<font color="red"><b>\u5c3c\u739b\u7adf\u7136\u8dea\u4e86\u4e86\uff0c\u4e0d\u8981\u544a\u8bc9\u6211\u4f60\u7684\u6c34\u8868\u5728\u91cc\u9762...</b></font>',//12
        ''
        ],
    btnClassArr=[
        {css:'icon-download',tag:'a',id:''},
        {css:'icon-btn-download',tag:'li',id:''},
        {css:'icon-btn-download',tag:'',id:''},
        {css:'download-btn',tag:'',id:''},
        {css:'',tag:'',id:'downFileButton'}
    ];
    try{
        downProxy = isOther ? disk.util.DownloadProxy || null : null;
        shareData = isOther ? disk.util.ViewShareUtils || null : null;
    }catch(e){}
    if(!isOther || (isOther && !FileUtils)){
        RestAPI=require("common:widget/restApi/restApi.js");
        Canvas=require("common:widget/canvasPanel/canvasPanel.js");
        Pancel=require("common:widget/panel/panel.js");
        Toast = require("common:widget/toast/toast.js");
        errorMsg = require("common:widget/errorMsg/errorMsg.js");
    }
    var helperMenuBtns=(function(){
        var menuTitleArr=['\u76f4\u63a5\u4e0b\u8f7d','\u590d\u5236\u94fe\u63a5','\u67e5\u770b\u94fe\u63a5'],panBtnsArr=[],html='';
        for(var i=0;i<btnClassArr.length;i++){
            var item=btnClassArr[i];
            var tmpItem=item.id!='' ? $('#'+item.id) : $('.'+item.css);
            var tmpArr=item.tag!='' ? tmpItem.parent(item.tag) : tmpItem;
            panBtnsArr=$.merge(panBtnsArr,tmpArr.toArray());
        }
        if(!panBtnsArr.length){return panBtnsArr;}
        html+='<div id="panHelperMenu" style="display:none;position:fixed;z-index:999999;">';
        html+='<ul class="pull-down-menu" style="display:block;margin:0px;padding:0px;left:0px;top:0px;list-style:none;">';
        for(var i=0;i<menuTitleArr.length;i++){
            html+='<li><a href="javascript:;" class="panHelperMenuBtn" type="'+i+'"><b>'+menuTitleArr[i]+'</b></a></li>';
        }
        html+='<li style="display:none;"><a href="' + getApiUrl('getnewversion', 1) + '" target="_blank">';
        html+='<img id="updateimg" title="\u6709\u4e00\u4efd\u7530" style="border:none;"/></a></li></ul></div>';
        $('<div>').html(html).appendTo(document.body);
        for (var i = 0; i < panBtnsArr.length; i++) {
            var item = panBtnsArr[i];
            createHelperBtn(item);
        }
        function createHelperBtn(btn) {
            var newnode=btn.cloneNode(true),html=newnode.innerHTML;
            $(newnode).attr('id','').attr('href','javascript:void(0)').attr('data-key','downLoadhelper').attr('node-type','btnHelper').attr('onclick','').css({width:63}).html(html.replace(/[\u4E00-\u9FA5]{2,4}(\(.*\)|\uff08.*\uff09)?/,'\u7f51\u76d8\u52a9\u624b')).unbind();
            var o=$('<div class="PanHelperBtn" style="display:inline-block;">').append(newnode)[0];
            btn.parentNode.insertBefore(o, btn.nextSibling);
            return o;
        }
        var helperBtn = $('.PanHelperBtn'),helperMenu = $('#panHelperMenu'),
        menuFun = function() {
            helperDownload($(this).attr('type') || 0);
            helperMenu.hide();
        };
        helperBtn.click(menuFun).mouseenter(function() {
            $(this).addClass('b-img-over');
            var o=$(this).children('a'),offset=o.offset(),w=o.outerWidth()-parseInt(o.css('paddingRight'));
            helperMenu.children('ul').css('width', w-2);
            helperMenu.css('top', offset.top + o.height() + parseInt(o.css('paddingTop')) - $(document).scrollTop());
            helperMenu.css('left', offset.left).show();
        }).mouseleave(function() {
            $(this).removeClass('b-img-over');
            helperMenu.hide();
        });
        $(document).scroll(function() {
            helperMenu.hide();
        });
        helperMenu.mouseenter(function() {
            $(this).show();
        }).mouseleave(function() {
            $(this).hide();
        });
        helperMenu.find('a').css('text-align', 'center');
        return helperMenu.find('a.panHelperMenuBtn').click(menuFun).toArray();
    })();
    if(!helperMenuBtns.length){return;}
    checkUpdate();
    function helperDownload(type){
        iframe=createDownloadIframe();
        iframe.src = 'javascript:;';
        var items = getListViewCheckedItems(),len = items.length;
        if(!len) {
            index = 1 == index ? 0 : 1;
            return myToast(msg[index]);
        }else if (len > 90) {
            return myToast(msg[2]);
        }
        if(1 == len) {
            var url = items[0].dlink;
            if(isUrl(url)) {
                if(2 == type) {
                    showHelperDialog(type, items, {"errno": 0,"dlink": url});
                }else if(1 == type){
                    copyText(url);
                }else{
                    myToast(msg[3],1);
                    iframe.src = url;
                }
            }else{
                getDownloadInfo(type, items);
            }
        }else {
            getDownloadInfo(type, items);
        }
        downloadCounter(items);
    }
    function getDownloadInfo(type, items, vcode) {
        if(!vcode) {
            showHelperDialog(helperMenuBtns.length+1, items);
            vcode = {};
        }
        var url = '',data = {},fidlist = '',fids = [];
        for (var i = 0; i < items.length; i++) {
            fids.push(items[i]['fs_id']);
        }
        fidlist = '[' + fids.join(',') + ']';
        if(isOther){
            if(FileUtils){
                url = disk.api.RestAPI.SHARE_GET_DLINK + '&uk=' + FileUtils.share_uk + '&shareid=' + FileUtils.share_id + '&timestamp=' + FileUtils.share_timestamp + '&sign=' + FileUtils.share_sign + '&fid_list=' + fidlist;
                data = {
                    shareid:FileUtils.share_id,
                    uk:FileUtils.share_uk,
                    fid_list:fidlist
                };
            }else{
                var context=yunData.getContext();
                if(typeof vcode =='object'){
                    url = '/api/sharedownload?' + 'uk=' + yunData.SHARE_UK + '&shareid=' + yunData.SHARE_ID + '&timestamp=' + yunData.TIMESTAMP + '&sign=' + yunData.SIGN + '&fid_list=' + fidlist;
                    data = 'encrypt=0&product=share&primaryid=' + yunData.SHARE_ID + '&shareid=' + yunData.SHARE_ID + '&uk=' + yunData.SHARE_UK + '&fid_list=' + fidlist+ '&extra=' + '{"sekey":"' + context.sekey + '"}';
                    data = {
                        encrypt:0,
                        extra:'{"sekey":"' + context.sekey + '"}',
                        product:'share',
                        primaryid:yunData.SHARE_ID,
                        shareid:yunData.SHARE_ID,
                        uk:yunData.SHARE_UK,
                        fid_list:fidlist
                    };
                }else{
                    url = RestAPI.GET_CAPTCHA + '?prod=share';
                }
            }
            if(typeof vcode =='object'){
                data = $.extend(data,vcode);
            }
            data.type=(items.length >1 || items[0]['isdir']) ? "batch" : "dlink";
        }else{
            url = RestAPI.DOWN_GET_DLINK;
            if ("function" != typeof yunData.sign2) try {
                yunData.sign2 = new Function("return " + yunData.sign2)();
            } catch (o) {}
            data={
                sign: base64Encode(yunData.sign2(yunData.sign3, yunData.sign1)),
                timestamp: yunData.timestamp,
                bdstoken: yunData.MYBDSTOKEN,
                fidlist: fidlist,
                type: (items.length >1 || items[0]['isdir']) ? "batch" : "dlink"
            };
        }
        httpHwnd = $.post(url, data,
            function(o) {
                var dlink = typeof o.dlink =='object' ? o.dlink[0]['dlink'] : o.dlink;
                if(-20 === o.errno){
                    getDownloadInfo(type, items, JSON.stringify(vcode) =='{}' ? 'getvcode' : 'showvcode');
                }else if (0 === o.errno) {
                    if(o.list || dlink){
                        if(!dlink){
                            var list = o.list,opt=list[0];
                            dlink=opt.dlink;
                        }
                        dlink = dlink + '&zipname=' + encodeURIComponent(getDownloadName(items));
                        o.dlink = dlink;
                        if (shareData) {
                            var obj = JSON.parse(shareData.viewShareData);
                            obj.dlink = dlink;
                            shareData.viewShareData = JSON.stringify(obj);
                        }
                        if (1 == items.length) {
                            items[0]['dlink'] = dlink;
                            if(items[0]['item']){
                                $(items[0]['item']).attr('dlink',dlink);
                            }
                            try{
                                if(yunData.SHAREPAGETYPE == "single_file_page"){
                                    yunData.FILEINFO = items;
                                }
                            }catch(e){}
                        }
                    }else{
                        if(o.vcode_img && o.vcode_str){
                            o.errno = -20;
                            if(vcode == 'getvcode'){
                                //return getDownloadInfo(type, items, getVCode('test',o.vcode_str));
                            }
                        }
                    }
                    showHelperDialog(type, items, o, vcode);
                }else{
                    showHelperDialog(type, items, o, vcode);
                }
            });
    }
    function showHelperDialog(type, items, opt, vcode) {
        var canvas =document.canvas ? document.canvas : Canvas ? new Canvas() : new disk.ui.Canvas(),
        _ = document.helperdialog || createHelperDialog(),isVisible = _.isVisible(),status=0;
        document.canvas = canvas;
        _.canvas = canvas;
        _.type = type;
        _.items = items;
        if (type < helperMenuBtns.length) {
            if (0 === opt.errno) {
                status=1;
                if(type < 2) {
                    _.canvas.setVisible(false);
                    _.setVisible(false);
                    if(0 == type){
                        iframe.src = opt.dlink;
                        myToast(msg[3],1);
                    } else {
                        copyText(opt.dlink);
                    }
                    return;
                }
                _.sharefilename.innerHTML = getDownloadName(items);
                _.sharedlink.value = opt.dlink;
                _.dlink = opt.dlink;
                //_.downloadbtn.href= opt.dlink;
                _.focusobj = _.sharedlink;
            } else if(-19 ==opt.errno) {
                status=2;
                _.vcodeimg.src = opt.img;
                _.vcodeimgsrc = opt.img;
                _.vcodevalue = opt.vcode;
                _.vcodetip.innerHTML = vcode ? msg[9] : '';
                _.vcodeinput.value = '';
                _.focusobj = _.vcodeinput;
            }else if(-20 ==opt.errno) {
                status=2;
                _.vcodeimg.src = opt.vcode_img;
                _.vcodeimgsrc = opt.vcode_img;
                _.vcodevalue = opt.vcode_str;
                _.vcodetip.innerHTML = vcode && vcode!='getvcode' ? msg[9] : '';
                _.vcodeinput.value = '';
                _.focusobj = _.vcodeinput;
            } else {
                _.canvas.setVisible(false);
                _.setVisible(false);
                return myToast(errorMsg ? errorMsg.ErrorMessage[opt.errno] : disk.util.shareErrorMessage[opt.errno] || (msg[11] + opt.errno));
            }
        }
        _.loading.style.display = 0==status ? '' : 'none';
        _.showdlink.style.display = 1==status ? '' : 'none';
        _.showvcode.style.display = 2==status ? '' : 'none';
        _.copytext.style.display = 1==status ? '' : 'none';
        if (!isVisible) {
            _.canvas.setVisible(true);
            _.setVisible(true);
        }
        _.setGravity(Pancel ? Pancel.CENTER : disk.ui.Panel.CENTER);
        _.focusobj.focus();
    }
    function createHelperDialog() {
        var html = '<div class="dlg-hd b-rlv"title="\u6709\u4e00\u4efd\u7530"><span title="\u5173\u95ed"id="helperdialogclose"class="dlg-cnr dlg-cnr-r"></span><h3><a href="'+getApiUrl('getnewversion',1)+'"target="_blank"style="color:#000;">'+APPNAME+'&nbsp;' + VERSION + '</a><a href="javascript:;"title="\u70b9\u6b64\u590d\u5236"id="copytext"style="float:right;margin-right:240px;display:none;">\u70b9\u6b64\u590d\u5236</a></h3></div><div class="download-mgr-dialog-msg center"id="helperloading"><b>\u6570\u636e\u8d76\u6765\u4e2d...</b></div><div id="showvcode"style="text-align:center;display:none;"><div class="dlg-bd download-verify"style="text-align:center;margin-top:25px;"><div class="verify-body">\u8bf7\u8f93\u5165\u9a8c\u8bc1\u7801\uff1a<input type="text"maxlength="4"class="input-code vcode"><img width="100"height="30"src=""alt="\u9a8c\u8bc1\u7801\u83b7\u53d6\u4e2d"class="img-code"><a class="underline"href="javascript:;">\u6362\u4e00\u5f20</a></div><div class="verify-error"style="text-align:left;margin-left:84px;"></div></div><br><div><div class="alert-dialog-commands clearfix"><a href="javascript:;"class="sbtn okay postvcode"><b>\u786e\u5b9a</b></a><a href="javascript:;"class="dbtn cancel"><b>\u5173\u95ed</b></a></div></div></div><div id="showdlink"style="text-align:center;display:none;"><div class="dlg-bd download-verify"><div style="padding:5px 0px;"><b><span id="sharefilename"></span></b></div><input type="text"name="sharedlink"id="sharedlink"class="input-code"maxlength="1024"value=""style="width:500px;border:1px solid #7FADDC;padding:3px;height:24px;"></div><br><div><div class="alert-dialog-commands clearfix"><a href="javascript:;"class="sbtn okay postdownload"><b>\u76f4\u63a5\u4e0b\u8f7d</b></a><a href="javascript:;"class="dbtn cancel"><b>\u5173\u95ed</b></a></div></div></div>',
        o=$('<div class="b-panel download-mgr-dialog helperdialog" style="width:550px;">').html(html).appendTo(document.body);
        o[0].pane = o[0];
        var _ = Pancel ? new Pancel(o[0]) : new disk.ui.Panel(o[0]),vcodeimg = o.find('img')[0],vcodeinput = o.find('.vcode')[0],
        sharedlink = o.find('#sharedlink')[0],vcodetip = o.find('.verify-error')[0],
        copytext= o.find('#copytext')[0],postdownloadBtn=o.find('.postdownload')[0],
        dialogClose = function() {
            vcodeinput.value = '';
            vcodetip.innerHTML = '';
            vcodeimg.src = '';
            _.canvas.setVisible(false);
            _.setVisible(false);
            if (httpHwnd) {httpHwnd.abort();}
        },
        postvcode = function() {
            if (httpHwnd) {httpHwnd.abort();}
            var v = vcodeinput.value,len = v.length,max = msg.length - 1,i = max,
            vcode = getVCode(v,_.vcodevalue);
            i = 0 == len ? 8 : (len < 4 ? 9 : i);
            vcodetip.innerHTML = msg[i];
            if (i != max) {return vcodeinput.focus();}
            getDownloadInfo(_.type, _.items, vcode);
        },
        postdownload = function(e) {
            //if(!e){iframe.src = _.dlink;}
            iframe.src = _.dlink;
            dialogClose();
            myToast(msg[3],1);
        };
        _._mUI.pane = o[0];
        _.loading = o.find('#helperloading')[0];
        _.showvcode = o.find('#showvcode')[0];
        _.showdlink = o.find('#showdlink')[0];
        _.copytext= copytext;
        _.downloadbtn=postdownloadBtn;
        _.vcodeinput = vcodeinput;
        _.sharedlink = sharedlink;
        _.sharefilename = o.find('#sharefilename')[0];
        _.vcodeimg = vcodeimg;
        _.vcodetip = vcodetip;
        _.vcodeimgsrc = '';
        _.vcodevalue = '';
        _.focusobj = sharedlink;
        $(copytext).click(function(){
            copyText(_.dlink);
            this.blur();
        });
        $(vcodeimg).siblings('a').click(function() {
            vcodeimg.src = _.vcodeimgsrc + '&' + new Date().getTime();
            vcodeinput.focus();
        });
        vcodeinput.onkeydown = function(e) {
            if (13 == e.keyCode) {postvcode();}
        };
        o.find('.postvcode').click(postvcode);
        $(postdownloadBtn).click(postdownload);
        $('#sharedlink').focusin(function() {
            this.style.boxShadow = '0 0 3px #7FADDC';
            this.select();
        }).focusout(function() {
            this.style.boxShadow = '';
        }).mouseover(function() {
            this.select();
            this.focus();
        }).keydown(function(e) {
            if (13 == e.keyCode) {postdownload();}
        });
        $(window).bind("resize",function() {
            _.setGravity(Pancel ? Pancel.CENTER : disk.ui.Panel.CENTER);
        });
        o.find('#helperdialogclose').click(dialogClose);
        o.find('.dbtn').click(dialogClose);
        _.setVisible(false);
        document.helperdialog = _;
        return _;
    }
    function getVCode(v,k){
        return FileUtils ? {input:v,vcode:_.k} : {vcode_input:v,vcode_str:k};
    }
    function myToast(msg, type) {
        try{
            unsafeWindow.myToastInjection(msg,type,isOther);
            return;
        }catch(e){}
        try {
            var Toast = {}, obtain,Pancel = null;
            if (isOther && disk.ui) {
                obtain = disk.ui.Toast;
                Toast.obtain = {};
                Toast.obtain.useToast = Utilities.useToast;
            } else {
                Toast = require("common:widget/toast/toast.js");
                Pancel=require("common:widget/panel/panel.js");
                obtain = Toast.obtain;
            }
            var o = Toast.obtain.useToast({
                toastMode:type ? obtain.MODE_SUCCESS :obtain.MODE_FAILURE,
                msg:msg,
                sticky:false,
                position:Pancel ? Pancel.TOP : (disk.ui ? disk.ui.Panel.TOP : undefined)
            });
            try {
                $(o._mUI.pane).css({
                    "z-index":999999
                });
            } catch (e) {}
        } catch (err) {
            if (!type) {
                alert(msg);
            }
        }
    }
    function copyText(text){
        GM_setClipboard(text);
        myToast(msg[10],1);
    }
    function createDownloadIframe(){
        if(iframe){return iframe;}
        var o = $('#helperdownloadiframe');
        iframe=o.length ? o[0] : '';
        if(!iframe) {
            iframe = $('<div style="display:none;">').html('<iframe src="" id="helperdownloadiframe" name="helperdownloadiframe"></iframe>').appendTo(document.body).find('#helperdownloadiframe')[0];
        }
        $(iframe).load(function(){
            if(this.src=='javascript:;'){return;}
            myToast(msg[12],0);
        });
        return iframe;
    }
    function getListViewCheckedItems(){
        var items=[];
        if(shareData){
            items.push(JSON.parse(shareData.viewShareData));
        }else if(isOther) {
            if(FileUtils){
                items = FileUtils.getListViewCheckedItems();
            }else if(yunData){
                if(yunData.SHAREPAGETYPE == "multi_file"){
                    items=getCheckItems();
                }else if(yunData.SHAREPAGETYPE == "single_file_page"){
                    items=yunData.FILEINFO;
                }
            }
        }else{
            items=getCheckItems();
        }
        return items;
    }
    function getCheckItems(){
        var items=[],boxCss=$('.list-selected').length ? 'module-list-view' : 'module-grid-view';
        $('div.' + boxCss).find('.item-active').each(function(i,o){
            items.push(getListViewCheckedItemInfo(o));
        });
        return items;
    }
    function getListViewCheckedItemInfo(obj){
        var o=$(obj),fs_id=o.attr('data-id'),category=o.attr('data-category'),
            isdir=o.attr('data-extname')=='dir' ? 1 : 0,
            server_filename=o.find('[node-type="name"]').attr('title'),
            dlink=o.attr('dlink') || '';
        return {'fs_id':fs_id,'category':category,'isdir':isdir,'server_filename':server_filename,'dlink':dlink,'item':obj};
    }
    function getDownloadName(items) {
        var packName=items[0]['server_filename'];
        if (items.length > 1 || 1 == items[0]['isdir']) {
            try{
                downProxy.prototype.setPackName(FileUtils.parseDirFromPath(items[0]['path']), !items[0]['isdir']);
                packName= downProxy.prototype._mPackName;
            }catch(e){
                packName='\u3010\u6279\u91cf\u4e0b\u8f7d\u3011'+packName+'\u7b49.zip';
            }
        }
        return packName;
    }
    function downloadCounter(C) { //C:items,B:isOneFile
        if (!isOther) {return;}
        var F = FileUtils ? FileUtils.share_uk || disk.util.ViewShareUtils.uk : yunData.SHARE_UK,
        D = FileUtils ? FileUtils.share_id : yunData.SHARE_ID,
        A = [],B = (1 == C.length && 0 == C[0].isdir),
        G = shareData ? disk.util.ViewShareUtils.albumId: '';
        for (var _ in C) {
            if (C.hasOwnProperty(_)) {
                var E = {
                    fid: C[_].fs_id,
                    category: C[_].category
                };
                A.push(E);
            }
        }
        G && B && $.post(disk.api.RestAPI.PCLOUD_ALBUM_DOWNLOAD_COUNTER, {
            uk: F,
            album_id: G,
            fs_id: C[_].fs_id
        });
        !G && $.post(FileUtils ? disk.api.RestAPI.MIS_COUNTER : RestAPI.MIS_COUNTER, {
            uk: F,
            filelist: JSON.stringify(A),
            sid: D,
            ctime: FileUtils ? FileUtils.share_ctime : yunData.SHARE_TIME,
            "public": FileUtils ? FileUtils.share_public_type : yunData.SHAREPAGETYPE,
            t: (new Date).getTime(),
            _: Math.random()
        });
        !G && B && $.get(FileUtils ? disk.api.RestAPI.SHARE_COUNTER : RestAPI.SHARE_COUNTER, {
            type: 1,
            shareid: D,
            uk: F,
            sign: FileUtils ? FileUtils.share_sign : yunData.SIGN,
            timestamp: FileUtils ? FileUtils.share_timestamp : yunData.TIMESTAMP,            
            t: new Date().getTime(),
            _: Math.random()
        });
    }
})();
loadJs('function myToastInjection(msg,type,isOther){try{var Toast={},obtain,Pancel=null;if(isOther&&disk.ui){obtain=disk.ui.Toast;Toast.obtain={};Toast.obtain.useToast=Utilities.useToast}else{Toast=require("common:widget/toast/toast.js");Pancel=require("common:widget/panel/panel.js");obtain=Toast.obtain}var o=Toast.obtain.useToast({toastMode:type?obtain.MODE_SUCCESS:obtain.MODE_FAILURE,msg:msg,sticky:false,position:Pancel?Pancel.TOP:(disk.ui?disk.ui.Panel.TOP:undefined)});try{$(o._mUI.pane).css({"z-index":999999})}catch(e){}}catch(err){if(!type){alert(msg)}}}');
function base64Encode(a){var b,c,d,e,f,g,h="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";for(d=a.length,c=0,b="";d>c;){if(e=255&a.charCodeAt(c++),c==d){b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4),b+="==";break}if(f=a.charCodeAt(c++),c==d){b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4|(240&f)>>4),b+=h.charAt((15&f)<<2),b+="=";break}g=a.charCodeAt(c++),b+=h.charAt(e>>2),b+=h.charAt((3&e)<<4|(240&f)>>4),b+=h.charAt((15&f)<<2|(192&g)>>6),b+=h.charAt(63&g)}return b;}
function isUrl(url) {
    return /^(http|https):\/\/([\w-]+(:[\w-]+)?@)?[\w-]+(\.[\w-]+)+(:[\d]+)?([#\/\?][^\s<>;"\']*)?$/.test(url);
}
function checkUpdate() {
    var js = 'var upinfo=document.getElementById("updateimg");';
    js += 'upinfo.src="' + getApiUrl('checkupdate', 1) + '";';
    js += 'upinfo.onload=function(){';
    js += 'upinfo.parentNode.parentNode.style.display="";';
    js += '}';
    loadJs(js);
}
function getApiUrl(action, type) {
    return 'http://app.duoluohua.com/update?action=' + action + '&system=script&appname=dupanlink&apppot=scriptjs&frompot=dupan&type=' + type + '&version=' + VERSION + '&t=' + t;
}
function loadJs(js) {
    var oHead = document.getElementsByTagName('HEAD')[0],
    oScript = document.createElement('script');
    oScript.type = 'text/javascript';
    oScript.text = js;
    oHead.appendChild(oScript);
}
function googleAnalytics() {
    var js = "var _gaq = _gaq || [];";
    js += "_gaq.push(['_setAccount', 'UA-43859764-1']);";
    js += "_gaq.push(['_trackPageview']);";
    js += "function googleAnalytics(){";
    js += "	var ga = document.createElement('script');ga.type = 'text/javascript';";
    js += "	ga.async = true;ga.src = 'https://ssl.google-analytics.com/ga.js';";
    js += "	var s = document.getElementsByTagName('script')[0];";
    js += "	s.parentNode.insertBefore(ga, s)";
    js += "}";
    js += "googleAnalytics();";
    js += "_gaq.push(['_trackEvent','dupanlink_script',String('" + VERSION + "')]);";
    loadJs(js);
}
googleAnalytics();


// ==UserScript==
// @name         百度网盘直接下载助手
// @namespace    undefined
// @version      0.9.24
// @description  直接下载百度网盘和百度网盘分享的文件,避免下载文件时调用百度网盘客户端,获取网盘文件的直接下载地址
// @author       ivesjay
// @match        *://pan.baidu.com/disk/home*
// @match        *://yun.baidu.com/disk/home*
// @match        *://pan.baidu.com/s/*
// @match        *://yun.baidu.com/s/*
// @match        *://pan.baidu.com/share/link*
// @match        *://yun.baidu.com/share/link*
// @require      https://code.jquery.com/jquery-latest.js
// @run-at       document-start
// @grant        unsafeWindow
// @grant        GM_setClipboard
// ==/UserScript==

(function() {
    'use strict';

    var $ = $ || window.$;
    var log_count = 1;
    var wordMapHttp = {
        'list-grid-switch':'yvgb9XJ',
        'list-switched-on':'ksbXZm',
        'grid-switched-on':'tch6W25',
        'list-switch':'lrbo9a',
        'grid-switch':'xh6poL',
        'checkbox':'EOGexf',
        'col-item':'Qxyfvg',
        'check':'fydGNC',
        'checked':'EzubGg',
        'list-view':'vdAfKMb',
        'item-active':'ngb9O6',
        'grid-view':'JKvHJMb',
        'bar-search':'OFaPaO',
        'default-dom':'xpX2PV',
        'bar':'qxnX2G5',
        'list-tools':'QDDOQB'
    };
    var wordMapHttps = {
        'list-grid-switch':'qobmXB1q',
        'list-switched-on':'ewXm1e',
        'grid-switched-on':'kxhkX2Em',
        'list-switch':'rvpXm63',
        'grid-switch':'mxgdJgwv',
        'checkbox':'EOGexf',
        'col-item':'Qxyfvg',
        'check':'fydGNC',
        'checked':'EzubGg',
        'list-view':'vdAfKMb',
        'item-active':'pcamXBRX',
        'grid-view':'JKvHJMb',
        'bar-search':'OFaPaO',
        'default-dom':'nyztJqWE',
        'bar':'mkseJqKQ',
        'list-tools':'QDDOQB'
    };
    var wordMap = location.protocol == 'http:' ? wordMapHttp : wordMapHttps;
    
    //console.log(wordMap);

    function slog(c1,c2,c3){
        c1 = c1?c1:'';
        c2 = c2?c2:'';
        c3 = c3?c3:'';
        console.log('#'+ log_count++ +'-BaiDuNetdiskHelper-log:',c1,c2,c3);
    }

    $(function(){
        switch(detectPage()){
            case 'disk':
                var panHelper = new PanHelper();
                panHelper.init();
                return;
            case 'share':
            case 's':
                var panShareHelper = new PanShareHelper();
                panShareHelper.init();
                return;
            default:
                return;
        }
    });

    //网盘页面的下载助手
    function PanHelper(){
        var yunData,sign,timestamp,bdstoken,logid,fid_list;
        var fileList=[],selectFileList=[],batchLinkList=[],batchLinkListAll=[],linkList=[],
            list_grid_status='list';
        var observer,currentPage,currentPath,currentCategory,dialog,searchKey;
        var panAPIUrl = location.protocol + "//" + location.host + "/api/";
        var restAPIUrl = location.protocol + "//pcs.baidu.com/rest/2.0/pcs/";
        var clientAPIUrl = location.protocol + "//d.pcs.baidu.com/rest/2.0/pcs/";

        this.init = function(){
            yunData = unsafeWindow.yunData;
            slog('yunData:',yunData);
            if(yunData === undefined){
                slog('页面未正常加载，或者百度已经更新！');
                return;
            }
            initParams();
            registerEventListener();
            createObserver();
            addButton();
            createIframe();
            dialog = new Dialog({addCopy:true});

            slog('网盘直接下载助手加载成功！');
        };

        function initParams(){
            sign = getSign();
            timestamp = getTimestamp();
            bdstoken = getBDStoken();
            logid = getLogID();
            currentPage = getCurrentPage();
            slog('Current display mode:',currentPage);

            if(currentPage == 'list')
                currentPath = getPath();

            if(currentPage == 'category')
                currentCategory = getCategory();

            if(currentPage == 'search')
                searchKey = getSearchKey();

            refreshListGridStatus();
            refreshFileList();
            refreshSelectList();
        }

        function refreshFileList(){
            if (currentPage == 'list') {
                fileList = getFileList();
            } else if (currentPage == 'category'){
                fileList = getCategoryFileList();
            } else if (currentPage == 'search') {
                fileList = getSearchFileList();
            }
        }

        function refreshSelectList(){
            selectFileList = [];
        }

        function refreshListGridStatus(){
            list_grid_status = getListGridStatus();
        }

        //获取当前的视图模式
        function getListGridStatus(){
            //return $('div.list-grid-switch').hasClass('list-switched-on')?'list':($('div.list-grid-switch').hasClass('grid-switched-on')?'grid':'list');
            //return $('div.itiWzPY').hasClass('kudtWY46')?'list':($('div.itiWzPY').hasClass('nytAL9w')?'grid':'list');
            return $('div.'+wordMap['list-grid-switch']).hasClass(wordMap['list-switched-on'])?'list':($('div.'+wordMap['list-grid-switch']).hasClass(wordMap['grid-switched-on'])?'grid':'list');
        }

        function registerEventListener(){
            registerHashChange();
            registerListGridStatus();
            registerCheckbox();
            registerAllCheckbox();
            registerFileSelect();
        }

        //监视地址栏#标签的变化
        function registerHashChange(){
            window.addEventListener('hashchange',function(e){
                refreshListGridStatus();
                if(getCurrentPage() == 'list') {
                    if(currentPage == getCurrentPage()){
                        if(currentPath == getPath()){
                            return;
                        } else {
                            currentPath = getPath();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        currentPath = getPath();
                        refreshFileList();
                        refreshSelectList();
                    }
                } else if (getCurrentPage() == 'category') {
                    if(currentPage == getCurrentPage()){
                        if(currentCategory == getCategory()){
                            return;
                        } else {
                            currentPage = getCurrentPage();
                            currentCategory = getCategory();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        currentCategory = getCategory();
                        refreshFileList();
                        refreshSelectList();
                    }
                } else if(getCurrentPage() == 'search') {
                    if(currentPage == getCurrentPage()){
                        if(searchKey == getSearchKey()){
                            return;
                        } else {
                            currentPage = getCurrentPage();
                            searchKey = getSearchKey();
                            refreshFileList();
                            refreshSelectList();
                        }
                    } else {
                        currentPage = getCurrentPage();
                        searchKey = getSearchKey();
                        refreshFileList();
                        refreshSelectList();
                    }
                }
            });
        }

        //监视视图变化
        function registerListGridStatus(){
            //var $a_list = $('a[node-type=list-switch]');
            //var $a_list = $('a[node-type=eepWzkk]');
            var $a_list = $('a[node-type='+wordMap['list-switch']+']');
            $a_list.click(function(){
                list_grid_status = 'list';
            });

            //var $a_grid = $('a[node-type=grid-switch]');
            //var $a_grid = $('a[node-type=ytnvWY7q]');
            var $a_grid = $('a[node-type='+wordMap['grid-switch']+']');
            $a_grid.click(function(){
                list_grid_status = 'grid';
            });
        }

        //文件选择框
        function registerCheckbox(){
            //var $checkbox = $('span.checkbox');
            //var $checkbox = $('span.EOGexf');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    var filename;
                    if(list_grid_status == 'list') {
                        //filename = $('div.file-name div.text a',$parent).attr('title');
                        filename = $('div.file-name div.text a',$parent).attr('title');
                    }else if(list_grid_status == 'grid'){
                        //filename = $('div.file-name a',$parent).attr('title');
                        filename = $('div.file-name a',$parent).attr('title');
                    }
                    //if($parent.hasClass('item-active')){
                    //if($parent.hasClass('prWzXA')){
                    if($parent.hasClass(wordMap['item-active'])){
                        slog('取消选中文件：'+filename);
                        for(var i=0;i<selectFileList.length;i++){
                            if(selectFileList[i].filename == filename){
                                selectFileList.splice(i,1);
                            }
                        }
                    }else{
                        slog('选中文件:'+filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterCheckbox(){
            //var $checkbox = $('span.checkbox');
            //var $checkbox = $('span.EOGexf');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //全选框
        function registerAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            //var $checkbox = $('div.Qxyfvg.fydGNC');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    //if($parent.hasClass('checked')){
                    //if($parent.hasClass('EzubGg')){
                    if($parent.hasClass(wordMap['checked'])){
                        slog('取消全选');
                        selectFileList = [];
                    } else {
                        slog('全部选中');
                        selectFileList = [];
                        $.each(fileList,function(index,element){
                            var obj = {
                                filename:element.server_filename,
                                path:element.path,
                                fs_id:element.fs_id,
                                isdir:element.isdir
                            };
                            selectFileList.push(obj);
                        });
                    }
                });
            });
        }

        function unregisterAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            //var $checkbox = $('div.Qxyfvg.fydGNC');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //单个文件选中，点击文件不是点击选中框，会只选中该文件
        function registerFileSelect(){
            //var $dd = $('div.list-view dd');
            //var $dd = $('div.vdAfKMb dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).bind('click',function(e){
                    var nodeName = e.target.nodeName.toLowerCase();
                    if(nodeName != 'span' && nodeName != 'a' && nodeName != 'em') {
                        slog('shiftKey:'+e.shiftKey);
                        if(!e.shiftKey){
                            selectFileList = [];
                            var filename = $('div.file-name div.text a',$(this)).attr('title');
                            slog('选中文件：' + filename);
                            $.each(fileList,function(index,element){
                                if(element.server_filename == filename){
                                    var obj = {
                                        filename:element.server_filename,
                                        path:element.path,
                                        fs_id:element.fs_id,
                                        isdir:element.isdir
                                    };
                                    selectFileList.push(obj);
                                }
                            });
                        }else{
                            selectFileList = [];
                            //var $dd_select = $('div.list-view dd.item-active');
                            //var $dd_select = $('div.vdAfKMb dd.prWzXA');
                            var $dd_select = $('div.'+wordMap['list-view']+' dd.'+wordMap['item-active']);
                            $.each($dd_select,function(index,element){
                                var filename = $('div.file-name div.text a',$(element)).attr('title');
                                slog('选中文件：' + filename);
                                $.each(fileList,function(index,element){
                                    if(element.server_filename == filename){
                                        var obj = {
                                            filename:element.server_filename,
                                            path:element.path,
                                            fs_id:element.fs_id,
                                            isdir:element.isdir
                                        };
                                        selectFileList.push(obj);
                                    }
                                });
                            });
                        }
                    }
                });
            });
        }

        function unregisterFileSelect(){
            //var $dd = $('div.list-view dd');
            //var $dd = $('div.vdAfKMb dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视文件列表显示变化
        function createObserver(){
            var MutationObserver = window.MutationObserver;
            var options = {
                'childList': true
            };
            observer = new MutationObserver(function(mutations){
                unregisterCheckbox();
                unregisterAllCheckbox();
                unregisterFileSelect();
                registerCheckbox();
                registerAllCheckbox();
                registerFileSelect();
            });
            
            //var list_view = document.querySelector('.list-view');
            //var grid_view = document.querySelector('.grid-view');
            
            //var list_view = document.querySelector('.vdAfKMb');
            //var grid_view = document.querySelector('.JKvHJMb');
            
            var list_view = document.querySelector('.'+wordMap['list-view']);
            var grid_view = document.querySelector('.'+wordMap['grid-view']);

            observer.observe(list_view,options);
            observer.observe(grid_view,options);
        }

        //添加助手按钮
        function addButton(){
            //$('div.bar-search').css('width','18%');//修改搜索框的宽度，避免遮挡
            //$('div.OFaPaO').css('width','18%');
            $('div.'+wordMap['bar-search']).css('width','18%');
            var $dropdownbutton = $('<span class="g-dropdown-button"></span>');
            var $dropdownbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><em class="icon icon-download" title="百度网盘下载助手"></em><span class="text" style="width: auto;">下载助手</span></span></a>');
            var $dropdownbutton_span = $('<span class="menu" style="width:96px"></span>');

            var $directbutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $directbutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $directbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">直接下载</span></span></a>');
            var $directbutton_menu = $('<span class="menu" style="width:120px;left:79px"></span>');
            var $directbutton_download_button = $('<a id="download-direct" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $directbutton_link_button = $('<a id="link-direct" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $directbutton_batchhttplink_button = $('<a id="batchhttplink-direct" class="g-button-menu" href="javascript:void(0);">批量链接(HTTP)</a>');
            var $directbutton_batchhttpslink_button = $('<a id="batchhttpslink-direct" class="g-button-menu" href="javascript:void(0);">批量链接(HTTPS)</a>');
            $directbutton_menu.append($directbutton_download_button).append($directbutton_link_button).append($directbutton_batchhttplink_button).append($directbutton_batchhttpslink_button);
            $directbutton.append($directbutton_span.append($directbutton_a).append($directbutton_menu));
            $directbutton.hover(function(){
                $directbutton_span.toggleClass('button-open');
            });
            $directbutton_download_button.click(downloadClick);
            $directbutton_link_button.click(linkClick);
            $directbutton_batchhttplink_button.click(batchClick);
            $directbutton_batchhttpslink_button.click(batchClick);

            var $apibutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $apibutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $apibutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">API下载</span></span></a>');
            var $apibutton_menu = $('<span class="menu" style="width:120px;left:77px"></span>');
            var $apibutton_download_button = $('<a id="download-api" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $apibutton_link_button = $('<a id="httplink-api" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $apibutton_batchhttplink_button = $('<a id="batchhttplink-api" class="g-button-menu" href="javascript:void(0);">批量链接(HTTP)</a>');
            var $apibutton_batchhttpslink_button = $('<a id="batchhttpslink-api" class="g-button-menu" href="javascript:void(0);">批量链接(HTTPS)</a>');
            $apibutton_menu.append($apibutton_download_button).append($apibutton_link_button).append($apibutton_batchhttplink_button).append($apibutton_batchhttpslink_button);
            $apibutton.append($apibutton_span.append($apibutton_a).append($apibutton_menu));
            $apibutton.hover(function(){
                $apibutton_span.toggleClass('button-open');
            });
            $apibutton_download_button.click(downloadClick);
            $apibutton_link_button.click(linkClick);
            $apibutton_batchhttplink_button.click(batchClick);
            $apibutton_batchhttpslink_button.click(batchClick);

            var $outerlinkbutton = $('<span class="g-button-menu" style="display:block"></span>');
            var $outerlinkbutton_span = $('<span class="g-dropdown-button g-dropdown-button-second" menulevel="2"></span>');
            var $outerlinkbutton_a = $('<a class="g-button" href="javascript:void(0);"><span class="g-button-right"><span class="text" style="width:auto">外链下载</span></span></a>');
            var $outerlinkbutton_menu = $('<span class="menu" style="width:120px;left:79px"></span>');
            var $outerlinkbutton_download_button = $('<a id="download-outerlink" class="g-button-menu" href="javascript:void(0);">下载</a>');
            var $outerlinkbutton_link_button = $('<a id="link-outerlink" class="g-button-menu" href="javascript:void(0);">显示链接</a>');
            var $outerlinkbutton_batchlink_button = $('<a id="batchlink-outerlink" class="g-button-menu" href="javascript:void(0);">批量链接</a>');
            $outerlinkbutton_menu.append($outerlinkbutton_download_button).append($outerlinkbutton_link_button).append($outerlinkbutton_batchlink_button);
            $outerlinkbutton.append($outerlinkbutton_span.append($outerlinkbutton_a).append($outerlinkbutton_menu));
            $outerlinkbutton.hover(function(){
                $outerlinkbutton_span.toggleClass('button-open');
            });
            $outerlinkbutton_download_button.click(downloadClick);
            $outerlinkbutton_link_button.click(linkClick);
            $outerlinkbutton_batchlink_button.click(batchClick);

            $dropdownbutton_span.append($directbutton).append($apibutton).append($outerlinkbutton);
            $dropdownbutton.append($dropdownbutton_a).append($dropdownbutton_span);

            $dropdownbutton.hover(function(){
                $dropdownbutton.toggleClass('button-open');
            });

            //$('div.default-dom div.bar div.list-tools').append($dropdownbutton);
            //$('div.irhW9pZ div.yqgR747 div.QDDOQB').append($dropdownbutton);
            $('div.'+wordMap['default-dom']+' div.'+wordMap['bar']+' div.'+wordMap['list-tools']).append($dropdownbutton);
        }

        //暂时没用
        // function addLoading(){
        //     var screenWidth = document.body.clientWidth;
        //     var screenHeight = document.body.scrollHeight;
        //     var left = (screenWidth-10)/2;
        //     var top = screenHeight/2;
        //     var $loading = $('<div id="dialog-loading" style="position:absolute;left:'+left+'px;top:'+top+'px;display:none;z-index:52;color:white;font-size:16px">处理中</div>');
        //     $('body').append($loading);
        // }

        function downloadClick(event){
            slog('选中文件列表：',selectFileList);
            var id = event.target.id;
            var downloadLink;

            if(id == 'download-direct'){
                var downloadType;
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length == 1) {
                    if (selectFileList[0].isdir === 1)
                        downloadType = 'batch';
                    else if (selectFileList[0].isdir === 0)
                        downloadType= 'dlink';
                    //downloadType = selectFileList[0].isdir==1?'batch':(selectFileList[0].isdir===0?'dlink':'batch');
                } else if(selectFileList.length > 1){
                    downloadType = 'batch';
                }
                fid_list = getFidList(selectFileList);
                var result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno === 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch'){
                        downloadLink = result.dlink;
                        if(selectFileList.length === 1)
                            downloadLink = downloadLink + '&zipname=' + encodeURIComponent(selectFileList[0].filename) + '.zip';
                    }
                    else{
                        alert("发生错误！");
                        return;
                    }
                } else if(result.errno == -1){
                    alert('文件不存在或已被百度和谐，无法下载！');
                    return;
                }else if(result.errno == 112){
                    alert("页面过期，请刷新重试！");
                    return;
                }else{
                    alert("发生错误！");
                    return;
                }
            }else{
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length > 1) {
                    alert("该方法不支持多文件下载！");
                    return;
                } else {
                    if(selectFileList[0].isdir == 1){
                        alert("该方法不支持目录下载！");
                        return;
                    }
                }
                if(id == 'download-api'){
                    downloadLink = getDownloadLinkWithRESTAPIBaidu(selectFileList[0].path);
                } else if (id == 'download-outerlink'){
                    var result = getDownloadLinkWithClientAPI(selectFileList[0].path);
                    if(result.errno == 0){
                        downloadLink = result.urls[0].url;
                    }else if(result.errno == 1){
                        alert('文件不存在！');
                        return;
                    }else if(result.errno == 2){
                        alert('文件不存在或者已被百度和谐，无法下载！');
                        return;
                    }else{
                        alert('发生错误！');
                        return;
                    }
                }
            }
            execDownload(downloadLink);
        }

        function linkClick(event){
            slog('选中文件列表：',selectFileList);
            var id = event.target.id;
            var linkList,tip;

            if(id.indexOf('direct') != -1){
                var downloadType;
                var downloadLink;
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length == 1) {
                    if (selectFileList[0].isdir === 1)
                        downloadType = 'batch';
                    else if (selectFileList[0].isdir === 0)
                        downloadType= 'dlink';
                } else if(selectFileList.length > 1){
                    downloadType = 'batch';
                }
                fid_list = getFidList(selectFileList);
                var result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno === 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch'){
                        slog(selectFileList);
                        downloadLink = result.dlink;
                        if(selectFileList.length === 1)
                            downloadLink = downloadLink + '&zipname=' + encodeURIComponent(selectFileList[0].filename) + '.zip';
                    }
                    else{
                        alert("发生错误！");
                        return;
                    }
                }else if(result.errno == -1){
                    alert('文件不存在或已被百度和谐，无法下载！');
                    return;
                }else if(result.errno == 112){
                    alert("页面过期，请刷新重试！");
                    return;
                }else{
                    alert("发生错误！");
                    return;
                }
                var httplink = downloadLink.replace(/^([A-Za-z]+):/,'http:');
                var httpslink = downloadLink.replace(/^([A-Za-z]+):/,'https:');
                var filename = '';
                $.each(selectFileList,function(index,element){
                    if(selectFileList.length == 1)
                        filename = element.filename;
                    else{
                        if(index ==0)
                            filename = element.filename;
                        else
                            filename = filename + ',' + element.filename;
                    }
                });
                linkList = {
                    filename:filename,
                    urls:[
                        {url:httplink,rank:1},
                        {url:httpslink,rank:2}
                    ]
                };
                tip = '显示模拟百度网盘网页获取的链接，可以使用右键迅雷下载，复制到下载工具需要传递cookie，多文件打包下载的链接可以直接复制使用';
                dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
            }else{
                if(selectFileList.length === 0) {
                    alert("获取选中文件失败，请刷新重试！");
                    return;
                } else if (selectFileList.length > 1) {
                    alert("该方法不支持多文件下载！");
                    return;
                } else {
                    if(selectFileList[0].isdir == 1){
                        alert("该方法不支持目录下载！");
                        return;
                    }
                }
                if(id.indexOf('api') != -1){
                    var downloadLink = getDownloadLinkWithRESTAPIBaidu(selectFileList[0].path);
                    var httplink = downloadLink.replace(/^([A-Za-z]+):/,'http:');
                    var httpslink = downloadLink.replace(/^([A-Za-z]+):/,'https:');
                    linkList = {
                        filename:selectFileList[0].filename,
                        urls:[
                            {url:httplink,rank:1},
                            {url:httpslink,rank:2}
                        ]
                    };
                    httplink = httplink.replace('250528','266719');
                    httpslink = httpslink.replace('250528','266719');
                    linkList.urls.push({url:httplink,rank:3});
                    linkList.urls.push({url:httpslink,rank:4});
                    tip = '显示模拟APP获取的链接(使用百度云ID)，可以使用右键迅雷下载，复制到下载工具需要传递cookie';
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
                } else if (id.indexOf('outerlink') != -1){
                    var result = getDownloadLinkWithClientAPI(selectFileList[0].path);
                    if(result.errno == 0){
                        linkList = {
                            filename:selectFileList[0].filename,
                            urls:result.urls
                        };
                    }else if(result.errno == 1){
                        alert('文件不存在！');
                        return;
                    }else if(result.errno == 2){
                        alert('文件不存在或者已被百度和谐，无法下载！');
                        return;
                    }else{
                        alert('发生错误！');
                        return;
                    }
                    tip = '显示模拟百度网盘客户端获取的链接，可以直接复制到下载工具使用，不需要cookie';
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip,showcopy:true,showedit:true});
                }
            }
            //dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
        }

        function batchClick(event){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('获取选中文件失败，请刷新重试！');
                return;
            }
            var id = event.target.id;
            var linkType,tip;
            linkType = id.indexOf('https') == -1 ? (id.indexOf('http') == -1 ? location.protocol+':' : 'http:') : 'https:';
            batchLinkList = [];
            batchLinkListAll = [];
            if(id.indexOf('direct') != -1){
                batchLinkList = getDirectBatchLink(linkType);
                tip = '显示所有选中文件的直接下载链接，文件夹显示为打包下载的链接';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }
                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
            } else if(id.indexOf('api') != -1){
                batchLinkList = getAPIBatchLink(linkType);
                tip = '显示所有选中文件的API下载链接，不显示文件夹';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }
                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
            } else if(id.indexOf('outerlink') != -1){
                batchLinkListAll = getOuterlinkBatchLinkAll();
                batchLinkList = getOuterlinkBatchLinkFirst(batchLinkListAll);
                tip = '显示所有选中文件的外部下载链接，不显示文件夹';
                if(batchLinkList.length === 0){
                    alert('没有链接可以显示，API链接不要全部选中文件夹！');
                    return;
                }

                dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true,alllist:batchLinkListAll,showall:true});
            }

            //dialog.open({title:'批量链接',type:'batch',list:batchLinkList,tip:tip,showcopy:true});
        }

        function getDirectBatchLink(linkType){
            var list = [];
            $.each(selectFileList,function(index,element){
                var downloadType,downloadLink,result;
                if(element.isdir == 0)
                    downloadType = 'dlink';
                else
                    downloadType = 'batch';
                fid_list = getFidList([element]);
                result = getDownloadLinkWithPanAPI(downloadType);
                if(result.errno == 0){
                    if(downloadType == 'dlink')
                        downloadLink = result.dlink[0].dlink;
                    else if(downloadType == 'batch')
                        downloadLink = result.dlink;
                    downloadLink = downloadLink.replace(/^([A-Za-z]+):/,linkType);
                }else{
                    downloadLink = 'error';
                }
                list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getAPIBatchLink(linkType){
            var list = [];
            $.each(selectFileList,function(index,element){
                if(element.isdir == 1)
                    return;
                var downloadLink;
                downloadLink = getDownloadLinkWithRESTAPIBaidu(element.path);
                downloadLink = downloadLink.replace(/^([A-Za-z]+):/,linkType);
                list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getOuterlinkBatchLinkAll(){
            var list = [];
            $.each(selectFileList,function(index,element){
                var result;
                if(element.isdir == 1)
                    return;
                result = getDownloadLinkWithClientAPI(element.path);
                if(result.errno == 0){
                    //downloadLink = result.urls[0].url;
                    list.push({filename:element.filename,links:result.urls});
                }else{
                    //downloadLink = 'error';
                    list.push({filename:element.filename,links:[{rank:1,url:'error'}]});
                }
                //list.push({filename:element.filename,downloadlink:downloadLink});
            });
            return list;
        }

        function getOuterlinkBatchLinkFirst(list){
            var result = [];
            $.each(list,function(index,element){
                result.push({filename:element.filename,downloadlink:element.links[0].url});
            });
            return result;
        }

        function getSign(){
            var signFnc;
            try{
                signFnc = new Function("return " + yunData.sign2)();
            } catch(e){
                throw new Error(e.message);
            }
            return base64Encode(signFnc(yunData.sign5,yunData.sign1));
        }

        //获取当前目录
        function getPath(){
            var hash = location.hash;
            var regx = /(^|&|\/)path=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取分类显示的类别，即地址栏中的type
        function getCategory(){
            var hash = location.hash;
            var regx = /(^|&|\/)type=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        function getSearchKey(){
            var hash = location.hash;
            var regx = /(^|&|\/)key=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取当前页面(list或者category)
        function getCurrentPage(){
            var hash = location.hash;
            return decodeURIComponent(hash.substring(hash.indexOf('#')+1,hash.indexOf('/')));
        }

        //获取文件列表
        function getFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + "list";
            var path = getPath();
            logid = getLogID();
            var params = {
                dir:path,
                bdstoken:bdstoken,
                logid:logid,
                order:'size',
                desc:0,
                clienttype:0,
                showempty:0,
                web:1,
                channel:'chunlei',
                appid:250528
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.list : [];
                }
            });
            return filelist;
        }

        //获取分类页面下的文件列表
        function getCategoryFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + "categorylist";
            var category = getCategory();
            logid = getLogID();
            var params = {
                category:category,
                bdstoken:bdstoken,
                logid:logid,
                order:'size',
                desc:0,
                clienttype:0,
                showempty:0,
                web:1,
                channel:'chunlei',
                appid:250528
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.info : [];
                }
            });
            return filelist;
        }

        function getSearchFileList(){
            var filelist = [];
            var listUrl = panAPIUrl + 'search';
            logid = getLogID();
            searchKey = getSearchKey();
            var params = {
                recursion:1,
                order:'time',
                desc:1,
                showempty:0,
                web:1,
                page:1,
                num:100,
                key:searchKey,
                channel:'chunlei',
                app_id:250528,
                bdstoken:bdstoken,
                logid:logid,
                clienttype:0
            };
            $.ajax({
                url:listUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    filelist = 0===response.errno ? response.list : [];
                }
            });
            return filelist;
        }

        //生成下载时的fid_list参数
        function getFidList(list){
            var fidlist = null;
            if (list.length === 0)
                return null;
            var fileidlist = [];
            $.each(list,function(index,element){
                fileidlist.push(element.fs_id);
            });
            fidlist = '[' + fileidlist + ']';
            return fidlist;
        }

        function getTimestamp(){
            return yunData.timestamp;
        }

        function getBDStoken(){
            return yunData.MYBDSTOKEN;
        }

        //获取直接下载地址
        //这个地址不是直接下载地址，访问这个地址会返回302，response header中的location才是真实下载地址
        //暂时没有找到提取方法
        function getDownloadLinkWithPanAPI(type){
            var downloadUrl = panAPIUrl + "download";
            var result;
            logid = getLogID();
            var params= {
                sign:sign,
                timestamp:timestamp,
                fidlist:fid_list,
                type:type,
                channel:'chunlei',
                web:1,
                app_id:250528,
                bdstoken:bdstoken,
                logid:logid,
                clienttype:0
            };
            $.ajax({
                url:downloadUrl,
                async:false,
                method:'GET',
                data:params,
                success:function(response){
                    result = response;
                }
            });
            return result;
        }

        function getDownloadLinkWithRESTAPIBaidu(path){
            var link = restAPIUrl + 'file?method=download&app_id=250528&path=' + encodeURIComponent(path);
            return link;
        }

        function getDownloadLinkWithRESTAPIES(path){
            var link = restAPIUrl + 'file?method=download&app_id=266719&path=' + encodeURIComponent(path);
            return link;
        }

        function getDownloadLinkWithClientAPI(path){
            var result;
            var url = clientAPIUrl + 'file?method=locatedownload&app_id=250528&ver=4.0&path=' + encodeURIComponent(path);
            $.ajax({
                url:url,
                method:'POST',
                xhrFields: {
                    withCredentials: true
                },
                async:false,
                success:function(response){
                    result = JSON.parse(response);
                },
                statusCode:{
                    404:function(response){
                        result = response;
                    }
                }
            });
            if(result){
                if(result.error_code == undefined){
                    if(result.urls == undefined){
                        result.errno = 2;
                    }else{
                        $.each(result.urls,function(index,element){
                            result.urls[index].url = element.url.replace('\\','');
                        });
                        result.errno = 0;
                    }
                }else if(result.error_code == 31066){
                    result.errno = 1;
                }else{
                    result.errno = -1;
                }
            }else{
                result = {};
                result.errno = -1;
            }
            return result;
        }

        function execDownload(link){
            slog("下载链接："+link);
            $('#helperdownloadiframe').attr('src',link);
        }

        function createIframe(){
            var $div = $('<div class="helper-hide" style="padding:0;margin:0;display:block"></div>');
            var $iframe = $('<iframe src="javascript:void(0)" id="helperdownloadiframe" style="display:none"></iframe>');
            $div.append($iframe);
            $('body').append($div);

        }
    }

    //分享页面的下载助手
    function PanShareHelper(){
        var yunData,sign,timestamp,bdstoken,channel,clienttype,web,app_id,logid,encrypt,product,uk,primaryid,fid_list,extra,shareid;
        var vcode;
        var shareType,buttonTarget,currentPath,list_grid_status,observer,dialog,vcodeDialog;
        var fileList=[],selectFileList=[];
        var panAPIUrl = location.protocol + "//" + location.host + "/api/";
        var shareListUrl = location.protocol + "//" + location.host + "/share/list";

        this.init = function(){
            yunData = unsafeWindow.yunData;
            slog('yunData:',yunData);
            if(yunData === undefined || yunData.FILEINFO == null){
                slog('页面未正常加载，或者百度已经更新！');
                return;
            }
            initParams();
            addButton();
            dialog = new Dialog({addCopy:false});
            vcodeDialog = new VCodeDialog(refreshVCode,confirmClick);
            createIframe();

            if(!isSingleShare()){
                registerEventListener();
                createObserver();
            }

            slog('分享直接下载加载成功!');
        };

        function initParams(){
            shareType = getShareType();
            sign = yunData.SIGN;
            timestamp = yunData.TIMESTAMP;
            bdstoken = yunData.MYBDSTOKEN;
            channel = 'chunlei';
            clienttype = 0;
            web = 1;
            app_id = 250528;
            logid = getLogID();
            encrypt = 0;
            product = 'share';
            primaryid = yunData.SHARE_ID;
            uk = yunData.SHARE_UK;

            if(shareType == 'secret'){
                extra = getExtra();
            }
            if(isSingleShare()){
                var obj = {};
                if(yunData.CATEGORY == 2){
                    obj.filename = yunData.FILENAME;
                    obj.path = yunData.PATH;
                    obj.fs_id = yunData.FS_ID;
                    obj.isdir = 0;
                } else {
                    obj.filename = yunData.FILEINFO[0].server_filename,
                        obj.path = yunData.FILEINFO[0].path,
                        obj.fs_id = yunData.FILEINFO[0].fs_id,
                        obj.isdir =yunData.FILEINFO[0].isdir
                }
                selectFileList.push(obj);
            } else {
                shareid = yunData.SHARE_ID;
                currentPath = getPath();
                list_grid_status = getListGridStatus();
                fileList = getFileList();
            }
        }

        //判断分享类型（public或者secret）
        function getShareType(){
            return yunData.SHARE_PUBLIC===1 ? 'public' : 'secret';
        }

        //判断是单个文件分享还是文件夹或者多文件分享
        function isSingleShare(){
            return yunData.getContext === undefined ? true : false;
        }

        //判断是否为自己的分享链接
        function isSelfShare(){
            return yunData.MYSELF == 1 ? true : false;
        }

        function getExtra(){
            var seKey = decodeURIComponent(getCookie('BDCLND'));
            return '{' + '"sekey":"' + seKey + '"' + "}";
        }

        //获取当前目录
        function getPath(){
            var hash = location.hash;
            var regx = /(^|&|\/)path=([^&]*)(&|$)/i;
            var result = hash.match(regx);
            return decodeURIComponent(result[2]);
        }

        //获取当前的视图模式
        function getListGridStatus(){
            var status = 'list';
            var $status_div = $('div.list-grid-switch');
            if ($status_div.hasClass('list-switched-on')){
                status = 'list';
            } else if ($status_div.hasClass('grid-switched-on')) {
                status = 'grid';
            }
            return status;
        }

        //添加下载助手按钮
        function addButton(){
            if(isSingleShare()){
                $('div.slide-show-right').css('width','500px');
                $('div.frame-main').css('width','96%');
                $('div.share-file-viewer').css('width','740px').css('margin-left','auto').css('margin-right','auto');
            }
            else
                $('div.slide-show-right').css('width','500px');
            var $dropdownbutton = $('<span class="g-dropdown-button"></span>');
            var $dropdownbutton_a = $('<a class="g-button" data-button-id="b200" data-button-index="200" href="javascript:void(0);"></a>');
            var $dropdownbutton_a_span = $('<span class="g-button-right"><em class="icon icon-download" title="百度网盘下载助手"></em><span class="text" style="width: auto;">下载助手</span></span>');
            var $dropdownbutton_span = $('<span class="menu" style="width:auto;z-index:41"></span>');

            var $downloadButton = $('<a data-menu-id="b-menu207" class="g-button-menu" href="javascript:void(0);">直接下载</a>');
            var $linkButton = $('<a data-menu-id="b-menu208" class="g-button-menu" href="javascript:void(0);">显示链接</a>');

            $dropdownbutton_span.append($downloadButton).append($linkButton);
            $dropdownbutton_a.append($dropdownbutton_a_span);
            $dropdownbutton.append($dropdownbutton_a).append($dropdownbutton_span);

            $dropdownbutton.hover(function(){
                $dropdownbutton.toggleClass('button-open');
            });

            $downloadButton.click(downloadButtonClick);
            $linkButton.click(linkButtonClick);

            $('div.module-share-top-bar div.bar div.button-box').append($dropdownbutton);
        }

        function createIframe(){
            var $div = $('<div class="helper-hide" style="padding:0;margin:0;display:block"></div>');
            var $iframe = $('<iframe src="javascript:void(0)" id="helperdownloadiframe" style="display:none"></iframe>');
            $div.append($iframe);
            $('body').append($div);
        }

        function registerEventListener(){
            registerHashChange();
            registerListGridStatus();
            registerCheckbox();
            registerAllCheckbox();
            registerFileSelect();
        }

        //监视地址栏#标签变化
        function registerHashChange(){
            window.addEventListener('hashchange',function(e){
                list_grid_status = getListGridStatus();
                if(currentPath == getPath()){
                    return;
                } else {
                    currentPath = getPath();
                    refreshFileList();
                    refreshSelectFileList();
                }
            });
        }

        function refreshFileList(){
            fileList = getFileList();
        }

        function refreshSelectFileList(){
            selectFileList = [];
        }

        //监视视图变化
        function registerListGridStatus(){
            var $a_list = $('a[node-type=list-switch]');
            $a_list.click(function(){
                list_grid_status = 'list';
            });

            var $a_grid = $('a[node-type=grid-switch]');
            $a_grid.click(function(){
                list_grid_status = 'grid';
            });
        }

        //监视文件选择框
        function registerCheckbox(){
            //var $checkbox = $('span.checkbox');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    var filename;
                    if(list_grid_status == 'list') {
                        filename = $('div.file-name div.text a',$parent).attr('title');
                    }else if(list_grid_status == 'grid'){
                        filename = $('div.file-name a',$parent).attr('title');
                    }
                    if($parent.hasClass('item-active')){
                        slog('取消选中文件：'+filename);
                        for(var i=0;i<selectFileList.length;i++){
                            if(selectFileList[i].filename == filename){
                                selectFileList.splice(i,1);
                            }
                        }
                    }else{
                        slog('选中文件：'+filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterCheckbox(){
            //var $checkbox = $('span.checkbox');
            var $checkbox = $('span.'+wordMap['checkbox']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视全选框
        function registerAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).bind('click',function(e){
                    var $parent = $(this).parent();
                    //if($parent.hasClass('checked')){
                    if($parent.hasClass(wordMap['checked'])){
                        slog('取消全选');
                        selectFileList = [];
                    } else {
                        slog('全部选中');
                        selectFileList = [];
                        $.each(fileList,function(index,element){
                            var obj = {
                                filename:element.server_filename,
                                path:element.path,
                                fs_id:element.fs_id,
                                isdir:element.isdir
                            };
                            selectFileList.push(obj);
                        });
                    }
                });
            });
        }

        function unregisterAllCheckbox(){
            //var $checkbox = $('div.col-item.check');
            var $checkbox = $('div.'+wordMap['col-item']+'.'+wordMap['check']);
            $checkbox.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视单个文件选中
        function registerFileSelect(){
            //var $dd = $('div.list-view dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).bind('click',function(e){
                    var nodeName = e.target.nodeName.toLowerCase();
                    if(nodeName != 'span' && nodeName != 'a' && nodeName != 'em') {
                        selectFileList = [];
                        var filename = $('div.file-name div.text a',$(this)).attr('title');
                        slog('选中文件：' + filename);
                        $.each(fileList,function(index,element){
                            if(element.server_filename == filename){
                                var obj = {
                                    filename:element.server_filename,
                                    path:element.path,
                                    fs_id:element.fs_id,
                                    isdir:element.isdir
                                };
                                selectFileList.push(obj);
                            }
                        });
                    }
                });
            });
        }

        function unregisterFileSelect(){
            //var $dd = $('div.list-view dd');
            var $dd = $('div.'+wordMap['list-view']+' dd');
            $dd.each(function(index,element){
                $(element).unbind('click');
            });
        }

        //监视文件列表显示变化
        function createObserver(){
            var MutationObserver = window.MutationObserver;
            var options = {
                'childList': true
            };
            observer = new MutationObserver(function(mutations){
                unregisterCheckbox();
                unregisterAllCheckbox();
                unregisterFileSelect();
                registerCheckbox();
                registerAllCheckbox();
                registerFileSelect(); 
            });
            //var list_view = document.querySelector('.list-view');
            //var grid_view = document.querySelector('.grid-view');
            
            var list_view = document.querySelector('.'+wordMap['list-view']);
            var grid_view = document.querySelector('.'+wordMap['grid-view']);

            observer.observe(list_view,options);
            observer.observe(grid_view,options);
        }

        //获取文件信息列表
        function getFileList(){
            var result = [];
            if(getPath() == '/'){
                result = yunData.FILEINFO;
            } else {
                logid = getLogID();
                var params = {
                    uk:uk,
                    shareid:shareid,
                    order:'other',
                    desc:1,
                    showempty:0,
                    web:web,
                    dir:getPath(),
                    t:Math.random(),
                    bdstoken:bdstoken,
                    channel:channel,
                    clienttype:clienttype,
                    app_id:app_id,
                    logid:logid
                };
                $.ajax({
                    url:shareListUrl,
                    method:'GET',
                    async:false,
                    data:params,
                    success:function(response){
                        if(response.errno === 0){
                            result = response.list;
                        }
                    }
                });
            }
            return result;
        }

        function downloadButtonClick(){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('获取文件ID失败，请重试');
                return;
            }
            buttonTarget = 'download';
            var downloadLink = getDownloadLink();

            if(downloadLink.errno == -20) {
                vcode = getVCode();
                if(vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open(vcode);
            } else if(downloadLink.errno == 112){
                alert('页面过期，请刷新重试');
                return;
            } else if (downloadLink.errno === 0) {
                var link;
                if(selectFileList.length == 1 && selectFileList[0].isdir === 0)
                    link = downloadLink.list[0].dlink;
                else
                    link = downloadLink.dlink;
                execDownload(link);
            } else {
                alert('获取下载链接失败！');
                return;
            }
        }

        //获取验证码
        function getVCode(){
            var url = panAPIUrl + 'getvcode';
            var result;
            logid = getLogID();
            var params = {
                prod:'pan',
                t:Math.random(),
                bdstoken:bdstoken,
                channel:channel,
                clienttype:clienttype,
                web:web,
                app_id:app_id,
                logid:logid
            };
            $.ajax({
                url:url,
                method:'GET',
                async:false,
                data:params,
                success:function(response){
                    result = response;
                }
            });
            return result;
        }

        //刷新验证码
        function refreshVCode(){
            vcode = getVCode();
            $('#dialog-img').attr('src',vcode.img);
        }

        //验证码确认提交
        function confirmClick(){
            var val = $('#dialog-input').val();
            if(val.length === 0) {
                $('#dialog-err').text('请输入验证码');
                return;
            } else if(val.length < 4) {
                $('#dialog-err').text('验证码输入错误，请重新输入');
                return;
            } 
            var result = getDownloadLinkWithVCode(val);
            if(result.errno == -20){
                vcodeDialog.close();
                $('#dialog-err').text('验证码输入错误，请重新输入');
                refreshVCode();
                if(!vcode || vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open();
            } else if (result.errno === 0) {
                vcodeDialog.close();
                var link;
                if(selectFileList.length ==1 && selectFileList[0].isdir === 0)
                    link = result.list[0].dlink;
                else
                    link = result.dlink;
                if(buttonTarget == 'download'){
                    execDownload(link);
                } else if (buttonTarget == 'link') {
                    var filename = '';
                    $.each(selectFileList,function(index,element){
                        if(selectFileList.length == 1)
                            filename = element.filename;
                        else{
                            if(index ==0)
                                filename = element.filename;
                            else
                                filename = filename + ',' + element.filename;
                        }
                    });
                    var linkList = {
                        filename:filename,
                        urls:[
                            {url:link,rank:1}
                        ]
                    };
                    var tip = "显示获取的链接，可以使用右键迅雷下载，复制无用，需要传递cookie";
                    dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
                }
            } else {
                alert('发生错误！');
                return;
            }
        }

        //生成下载用的fid_list参数
        function getFidList(){
            var fidlist = [];
            $.each(selectFileList,function(index,element){
                fidlist.push(element.fs_id);
            });
            return '[' + fidlist + ']';
        }

        function linkButtonClick(){
            slog('选中文件列表：',selectFileList);
            if(selectFileList.length === 0){
                alert('没有选中文件，请重试');
                return;
            }
            buttonTarget = 'link';
            var downloadLink = getDownloadLink();

            if(downloadLink.errno == -20) {
                vcode = getVCode();
                if(!vcode || vcode.errno !== 0){
                    alert('获取验证码失败！');
                    return;
                }
                vcodeDialog.open(vcode);
            } else if (downloadLink.errno == 112) {
                alert('页面过期，请刷新重试');
                return;
            } else if (downloadLink.errno === 0) {
                var link;
                if(selectFileList.length == 1 && selectFileList[0].isdir === 0)
                    link = downloadLink.list[0].dlink;
                else
                    link = downloadLink.dlink;
                if(selectFileList.length == 1)
                    $('#dialog-downloadlink').attr('href',link).text(link);
                else
                    $('#dialog-downloadlink').attr('href',link).text(link);
                var filename = '';
                $.each(selectFileList,function(index,element){
                    if(selectFileList.length == 1)
                        filename = element.filename;
                    else{
                        if(index ==0)
                            filename = element.filename;
                        else
                            filename = filename + ',' + element.filename;
                    }
                });
                var linkList = {
                    filename:filename,
                    urls:[
                        {url:link,rank:1}
                    ]
                };
                var tip = "显示获取的链接，可以使用右键迅雷下载，复制无用，需要传递cookie";
                dialog.open({title:'下载链接',type:'link',list:linkList,tip:tip});
            } else {
                alert('获取下载链接失败！');
                return;
            }
        }

        //获取下载链接
        function getDownloadLink(){
            var result;
            if(isSingleShare){
                fid_list = getFidList();
                logid = getLogID();
                var url = panAPIUrl + 'sharedownload?sign=' + sign + '&timestamp=' + timestamp + '&bdstoken=' + bdstoken + '&channel=' + channel + '&clienttype=' + clienttype + '&web='+ web + '&app_id=' + app_id + '&logid=' + logid;
                var params = {
                    encrypt:encrypt,
                    product:product,
                    uk:uk,
                    primaryid:primaryid,
                    fid_list:fid_list
                };
                if(shareType == 'secret'){
                    params.extra = extra;
                }
                if(selectFileList[0].isdir == 1 || selectFileList.length > 1){
                    params.type = 'batch';
                }
                $.ajax({
                    url:url,
                    method:'POST',
                    async:false,
                    data:params,
                    success:function(response){
                        result = response;
                    }
                });
            }
            return result;
        }

        //有验证码输入时获取下载链接
        function getDownloadLinkWithVCode(vcodeInput){
            var result;
            if(isSingleShare){
                fid_list = getFidList();
                var url = panAPIUrl + 'sharedownload?sign=' + sign + '&timestamp=' + timestamp + '&bdstoken=' + bdstoken + '&channel=' + channel + '&clienttype=' + clienttype + '&web='+ web + '&app_id=' + app_id + '&logid=' + logid;
                var params = {
                    encrypt:encrypt,
                    product:product,
                    vcode_input:vcodeInput,
                    vcode_str:vcode.vcode,
                    uk:uk,
                    primaryid:primaryid,
                    fid_list:fid_list
                };
                if(shareType == 'secret'){
                    params.extra = extra;
                }
                if(selectFileList[0].isdir == 1 || selectFileList.length > 1 ){
                    params.type = 'batch';
                }
                $.ajax({
                    url:url,
                    method:'POST',
                    async:false,
                    data:params,
                    success:function(response){
                        result = response;
                    }
                });
            }
            return result;
        }

        function execDownload(link){
            slog('下载链接：'+link);
            $('#helperdownloadiframe').attr('src',link);
        }
    }

    function base64Encode(t){
        var a, r, e, n, i, s, o = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
        for (e = t.length,r = 0,a = ""; e > r; ) {
            if (n = 255 & t.charCodeAt(r++),r == e) {
                a += o.charAt(n >> 2);
                a += o.charAt((3 & n) << 4);
                a += "==";
                break;
            }
            if (i = t.charCodeAt(r++),r == e) {
                a += o.charAt(n >> 2);
                a += o.charAt((3 & n) << 4 | (240 & i) >> 4);
                a += o.charAt((15 & i) << 2);
                a += "=";
                break;
            }
            s = t.charCodeAt(r++);
            a += o.charAt(n >> 2);
            a += o.charAt((3 & n) << 4 | (240 & i) >> 4);
            a += o.charAt((15 & i) << 2 | (192 & s) >> 6);
            a += o.charAt(63 & s);
        }
        return a;
    }

    function detectPage(){
        var regx = /[\/].+[\/]/g;
        var page = location.pathname.match(regx);
        return page[0].replace(/\//g,'');
    }

    function getCookie(e) {
        var o, t;
        var n = document,c=decodeURI;
        return n.cookie.length > 0 && (o = n.cookie.indexOf(e + "="),-1 != o) ? (o = o + e.length + 1,t = n.cookie.indexOf(";", o),-1 == t && (t = n.cookie.length),c(n.cookie.substring(o, t))) : "";
    }

    function getLogID(){
        var name = "BAIDUID";
        var u = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/~！@#￥%……&";
        var d = /[\uD800-\uDBFF][\uDC00-\uDFFFF]|[^\x00-\x7F]/g;
        var f = String.fromCharCode;
        function l(e){
            if (e.length < 2) {
                var n = e.charCodeAt(0);
                return 128 > n ? e : 2048 > n ? f(192 | n >>> 6) + f(128 | 63 & n) : f(224 | n >>> 12 & 15) + f(128 | n >>> 6 & 63) + f(128 | 63 & n);
            }
            var n = 65536 + 1024 * (e.charCodeAt(0) - 55296) + (e.charCodeAt(1) - 56320);
            return f(240 | n >>> 18 & 7) + f(128 | n >>> 12 & 63) + f(128 | n >>> 6 & 63) + f(128 | 63 & n);
        }
        function g(e){
            return (e + "" + Math.random()).replace(d, l);
        }
        function m(e){
            var n = [0, 2, 1][e.length % 3];
            var t = e.charCodeAt(0) << 16 | (e.length > 1 ? e.charCodeAt(1) : 0) << 8 | (e.length > 2 ? e.charCodeAt(2) : 0);
            var o = [u.charAt(t >>> 18), u.charAt(t >>> 12 & 63), n >= 2 ? "=" : u.charAt(t >>> 6 & 63), n >= 1 ? "=" : u.charAt(63 & t)];
            return o.join("");
        }
        function h(e){
            return e.replace(/[\s\S]{1,3}/g, m);
        }
        function p(){
            return h(g((new Date()).getTime()));
        }
        function w(e,n){
            return n ? p(String(e)).replace(/[+\/]/g, function(e) {
                return "+" == e ? "-" : "_";
            }).replace(/=/g, "") : p(String(e));
        }
        return w(getCookie(name));
    }

    function Dialog(){
        var linkList = [];
        var showParams;
        var dialog,shadow;
        function createDialog(){
            var screenWidth = document.body.clientWidth;
            var dialogLeft = screenWidth>800 ? (screenWidth-800)/2 : 0;
            var $dialog_div = $('<div class="dialog" style="width: 800px; top: 0px; bottom: auto; left: '+dialogLeft+'px; right: auto; display: hidden; visibility: visible; z-index: 52;"></div>');
            var $dialog_header = $('<div class="dialog-header"><h3><span class="dialog-title" style="display:inline-block;width:740px;white-space:nowrap;overflow-x:hidden;text-overflow:ellipsis"></span></h3></div>');
            var $dialog_control = $('<div class="dialog-control"><span class="dialog-icon dialog-close">×</span></div>');
            var $dialog_body = $('<div class="dialog-body" style="max-height:450px;overflow-y:auto;padding:0 20px;"></div>');
            var $dialog_tip = $('<div class="dialog-tip" style="padding-left:20px;background-color:#faf2d3;border-top: 1px solid #c4dbfe;"><p></p></div>');

            $dialog_div.append($dialog_header.append($dialog_control)).append($dialog_body);

            //var $dialog_textarea = $('<textarea class="dialog-textarea" style="display:none;width"></textarea>');
            var $dialog_radio_div = $('<div class="dialog-radio" style="display:none;width:760px;padding-left:20px;padding-right:20px"></div>');
            var $dialog_radio_multi = $('<input type="radio" name="showmode" checked="checked" value="multi"><span>多行</span>');
            var $dialog_radio_single = $('<input type="radio" name="showmode" value="single"><span>单行</span>');
            $dialog_radio_div.append($dialog_radio_multi).append($dialog_radio_single);
            $dialog_div.append($dialog_radio_div);
            $('input[type=radio][name=showmode]',$dialog_radio_div).change(function(){
                var value = this.value;
                var $textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var content = $textarea.val();
                if(value == 'multi'){
                    content = content.replace(/\s+/g,'\n');
                    $textarea.css('height','300px');
                } else if(value == 'single'){
                    content = content.replace(/\n+/g,' ');
                    $textarea.css('height','');
                }
                $textarea.val(content);
            });

            var $dialog_button = $('<div class="dialog-button" style="display:none"></div>');
            var $dialog_button_div = $('<div style="display:table;margin:auto"></div>')
            var $dialog_copy_button = $('<button id="dialog-copy-button" style="display:none">复制</button>');
            var $dialog_edit_button = $('<button id="dialog-edit-button" style="display:none">编辑</button>');
            var $dialog_exit_button = $('<button id="dialog-exit-button" style="display:none">退出</button>');

            $dialog_button_div.append($dialog_copy_button).append($dialog_edit_button).append($dialog_exit_button);
            $dialog_button.append($dialog_button_div);
            $dialog_div.append($dialog_button);

            $dialog_copy_button.click(function(){
                var content = '';
                if(showParams.type == 'batch'){
                    $.each(linkList,function(index,element){
                        if(element.downloadlink == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.downloadlink;
                        else
                            content =  content + element.downloadlink + '\n';
                    });
                } else if(showParams.type == 'link'){
                    $.each(linkList,function(index,element){
                        if(element.url == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.url;
                        else
                            content =  content + element.url + '\n';
                    });
                }
                GM_setClipboard(content,'text');
                alert('已将链接复制到剪贴板！');
            });

            $dialog_edit_button.click(function(){
                var $dialog_textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var $dialog_item = $('div.dialog-body div',dialog);
                $dialog_item.hide();
                $dialog_copy_button.hide();
                $dialog_edit_button.hide();
                $dialog_textarea.show();
                $dialog_radio_div.show();
                $dialog_exit_button.show();
            });

            $dialog_exit_button.click(function(){
                var $dialog_textarea = $('div.dialog-body textarea[name=dialog-textarea]',dialog);
                var $dialog_item = $('div.dialog-body div',dialog);
                $dialog_textarea.hide();
                $dialog_radio_div.hide();
                $dialog_item.show();
                $dialog_exit_button.hide();
                $dialog_copy_button.show();
                $dialog_edit_button.show();
            });

            $dialog_div.append($dialog_tip);
            $('body').append($dialog_div);
            $dialog_div.dialogDrag();
            $dialog_control.click(dialogControl);
            return $dialog_div;
        }

        function createShadow(){
            var $shadow = $('<div class="dialog-shadow" style="position: fixed; left: 0px; top: 0px; z-index: 50; background: rgb(0, 0, 0) none repeat scroll 0% 0%; opacity: 0.5; width: 100%; height: 100%; display: none;"></div>');
            $('body').append($shadow);
            return $shadow;
        }

        this.open = function(params){
            showParams = params;
            linkList = [];
            if(params.type == 'link'){
                linkList = params.list.urls;
                $('div.dialog-header h3 span.dialog-title',dialog).text(params.title + "：" +params.list.filename);
                $.each(params.list.urls,function(index,element){
                    var $div = $('<div><div style="width:30px;float:left">'+element.rank+':</div><div style="white-space:nowrap;overflow:hidden;text-overflow:ellipsis"><a href="'+element.url+'">'+element.url+'</a></div></div>');
                    $('div.dialog-body',dialog).append($div);
                });
            } else if(params.type == 'batch'){
                linkList = params.list;
                $('div.dialog-header h3 span.dialog-title',dialog).text(params.title);
                if(params.showall){
                    $.each(params.list,function(index,element){
                        var $item_div = $('<div class="item-container" style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap"></div>');
                        var $item_name = $('<div style="width:100px;float:left;overflow:hidden;text-overflow:ellipsis" title="'+element.filename+'">'+element.filename+'</div>');
                        var $item_sep = $('<div style="width:12px;float:left"><span>：</span></div>');
                        var $item_link_div = $('<div class="item-link" style="float:left;width:618px;"></div>');
                        var $item_first = $('<div class="item-first" style="overflow:hidden;text-overflow:ellipsis"><a href="'+element.downloadlink+'">'+element.downloadlink+'</a></div>');
                        $item_link_div.append($item_first);
                        $.each(params.alllist[index].links,function(n,item){
                            if(element.downloadlink == item.url)
                                return;
                            var $item = $('<div class="item-ex" style="display:none;overflow:hidden;text-overflow:ellipsis"><a href="'+item.url+'">'+item.url+'</a></div>');
                            $item_link_div.append($item);
                        });
                        var $item_ex = $('<div style="width:15px;float:left;cursor:pointer;text-align:center;font-size:16px"><span>+</span></div>');
                        $item_div.append($item_name).append($item_sep).append($item_link_div).append($item_ex);
                        $item_ex.click(function(){
                            var $parent = $(this).parent();
                            $parent.toggleClass('showall');
                            if($parent.hasClass('showall')){
                                $(this).text('-');
                                $('div.item-link div.item-ex',$parent).show();
                            } else {
                                $(this).text('+');
                                $('div.item-link div.item-ex',$parent).hide();
                            }
                        });
                        $('div.dialog-body',dialog).append($item_div);
                    });
                }else{
                    $.each(params.list,function(index,element){
                        var $div = $('<div style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap"><div style="width:100px;float:left;overflow:hidden;text-overflow:ellipsis" title="'+element.filename+'">'+element.filename+'</div><span>：</span><a href="'+element.downloadlink+'">'+element.downloadlink+'</a></div>');
                        $('div.dialog-body',dialog).append($div);
                    });
                }
            }

            if(params.tip){
                $('div.dialog-tip p',dialog).text(params.tip);
            }

            if(params.showcopy){
                $('div.dialog-button',dialog).show();
                $('div.dialog-button button#dialog-copy-button',dialog).show();
            }
            if(params.showedit){
                $('div.dialog-button',dialog).show();
                $('div.dialog-button button#dialog-edit-button',dialog).show();
                var $dialog_textarea = $('<textarea name="dialog-textarea" style="display:none;resize:none;width:758px;height:300px;white-space:pre;word-wrap:normal;overflow-x:scroll"></textarea>');
                var content = '';
                if(showParams.type == 'batch'){
                    $.each(linkList,function(index,element){
                        if(element.downloadlink == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.downloadlink;
                        else
                            content =  content + element.downloadlink + '\n';
                    });
                } else if(showParams.type == 'link'){
                    $.each(linkList,function(index,element){
                        if(element.url == 'error')
                            return;
                        if(index == linkList.length-1)
                            content = content + element.url;
                        else
                            content =  content + element.url + '\n';
                    });
                }
                $dialog_textarea.val(content);
                $('div.dialog-body',dialog).append($dialog_textarea);
            }

            shadow.show();
            dialog.show();
        }

        this.close = function(){
            dialogControl();
        }

        function dialogControl(){
            $('div.dialog-body',dialog).children().remove();
            $('div.dialog-header h3 span.dialog-title',dialog).text('');
            $('div.dialog-tip p',dialog).text('');
            $('div.dialog-button',dialog).hide();
            $('div.dialog-radio input[type=radio][name=showmode][value=multi]',dialog).prop('checked',true);
            $('div.dialog-radio',dialog).hide();
            $('div.dialog-button button#dialog-copy-button',dialog).hide();
            $('div.dialog-button button#dialog-edit-button',dialog).hide();
            $('div.dialog-button button#dialog-exit-button',dialog).hide();
            dialog.hide();
            shadow.hide();
        }

        dialog = createDialog();
        shadow = createShadow();
    }

    function VCodeDialog(refreshVCode,confirmClick){
        var dialog,shadow;
        function createDialog(){
            var screenWidth = document.body.clientWidth;
            var dialogLeft = screenWidth>520 ? (screenWidth-520)/2 : 0;
            var $dialog_div = $('<div class="dialog" id="dialog-vcode" style="width:520px;top:0px;bottom:auto;left:'+dialogLeft+'px;right:auto;display:none;visibility:visible;z-index:52"></div>');
            var $dialog_header = $('<div class="dialog-header"><h3><span class="dialog-header-title"><em class="select-text">提示</em></span></h3></div>');
            var $dialog_control = $('<div class="dialog-control"><span class="dialog-icon dialog-close icon icon-close"><span class="sicon">x</span></span></div>');
            var $dialog_body = $('<div class="dialog-body"></div>');
            var $dialog_body_div = $('<div style="text-align:center;padding:22px"></div>');
            var $dialog_body_download_verify = $('<div class="download-verify" style="margin-top:10px;padding:0 28px;text-align:left;font-size:12px;"></div>');
            var $dialog_verify_body = $('<div class="verify-body">请输入验证码：</div>');
            var $dialog_input = $('<input id="dialog-input" type="text" style="padding:3px;width:85px;height:23px;border:1px solid #c6c6c6;background-color:white;vertical-align:middle;" class="input-code" maxlength="4">');
            var $dialog_img = $('<img id="dialog-img" class="img-code" style="margin-left:10px;vertical-align:middle;" alt="点击换一张" src="" width="100" height="30">');
            var $dialog_refresh = $('<a href="javascript:void(0)" style="text-decoration:underline;" class="underline">换一张</a>');
            var $dialog_err = $('<div id="dialog-err" style="padding-left:84px;height:18px;color:#d80000" class="verify-error"></div>');
            var $dialog_footer = $('<div class="dialog-footer g-clearfix"></div>');
            var $dialog_confirm_button = $('<a class="g-button g-button-blue" data-button-id="" data-button-index href="javascript:void(0)" style="padding-left:36px"><span class="g-button-right" style="padding-right:36px;"><span class="text" style="width:auto;">确定</span></span></a>');
            var $dialog_cancel_button = $('<a class="g-button" data-button-id="" data-button-index href="javascript:void(0);" style="padding-left: 36px;"><span class="g-button-right" style="padding-right: 36px;"><span class="text" style="width: auto;">取消</span></span></a>');

            $dialog_header.append($dialog_control);
            $dialog_verify_body.append($dialog_input).append($dialog_img).append($dialog_refresh);
            $dialog_body_download_verify.append($dialog_verify_body).append($dialog_err);
            $dialog_body_div.append($dialog_body_download_verify);
            $dialog_body.append($dialog_body_div);
            $dialog_footer.append($dialog_confirm_button).append($dialog_cancel_button);
            $dialog_div.append($dialog_header).append($dialog_body).append($dialog_footer);
            $('body').append($dialog_div);

            $dialog_div.dialogDrag();

            $dialog_control.click(dialogControl);
            $dialog_img.click(refreshVCode);
            $dialog_refresh.click(refreshVCode);
            $dialog_input.keypress(function(event){
                if(event.which == 13)
                    confirmClick();
            });
            $dialog_confirm_button.click(confirmClick);
            $dialog_cancel_button.click(dialogControl);
            $dialog_input.click(function(){
                $('#dialog-err').text('');
            });
            return $dialog_div;
        }
        this.open = function(vcode){
            if(vcode)
                $('#dialog-img').attr('src',vcode.img);
            dialog.show();
            shadow.show();
        }
        this.close = function(){
            dialogControl();
        }
        dialog = createDialog();
        shadow = $('div.dialog-shadow');
        function dialogControl(){
            $('#dialog-img',dialog).attr('src','');
            $('#dialog-err').text('');
            dialog.hide();
            shadow.hide();
        }
    }

    $.fn.dialogDrag = function(){
        var mouseInitX,mouseInitY,dialogInitX,dialogInitY;
        var screenWidth = document.body.clientWidth;
        var $parent = this;
        $('div.dialog-header',this).mousedown(function(event){
            mouseInitX = parseInt(event.pageX);
            mouseInitY = parseInt(event.pageY);
            dialogInitX = parseInt($parent.css('left').replace('px',''));
            dialogInitY = parseInt($parent.css('top').replace('px',''));
            $(this).mousemove(function(event){
                var tempX = dialogInitX + parseInt(event.pageX) - mouseInitX;
                var tempY = dialogInitY + parseInt(event.pageY) - mouseInitY;
                var width = parseInt($parent.css('width').replace('px',''));
                tempX = tempX<0 ? 0 : tempX>screenWidth-width ? screenWidth-width : tempX;
                tempY = tempY<0 ? 0 : tempY;
                $parent.css('left',tempX+'px').css('top',tempY+'px');
            });
        });
        $('div.dialog-header',this).mouseup(function(event){
            $(this).unbind('mousemove');
        });
    }

})();
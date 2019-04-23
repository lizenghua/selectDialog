# selectDialog
select带选择搜索的弹窗效果

![预览](https://github.com/lizenghua/selectDialog/blob/master/%E6%95%88%E6%9E%9C%E9%A2%84%E8%A7%88.png)

## 核心逻辑

```javascript
/**
/* 弹窗选择插件
 * 依赖组件 ：jquery artDialog bootstrap
 * 使用方式 ：$('#id').selectDialog({配置项});
 * 取值方式 ：$('#id').val();
 * 可用配置项 ：弹窗的宽：width, 弹窗的高：height, 弹窗的标题：title, 确定按钮的文字:okVal,关闭按钮的文字：cancelVal, 无选择时显示的文字:emptyText
 */
;(function($) {
    $.fn.selectDialog=function(options) {
        var defaults= {
            width: 800, height: 'auto', title: '选择', okValue: '保存', cancelValue: '取消', emptyText: ''
        }
        var options=$.extend(defaults, options);
        var values=[];
        var optionText=[];
        var select=this;
        var module='';
        if(select.attr('multiple')==='multiple') {
            //多选
            module='multiple';
        }
        else {
            module='single';
        }
        options.selected=[]; //选中的行
        options.emptyText=options.emptyText?options.emptyText: select.attr('placeholder'); //无选择时显示的文字
        select.find('option').each(function(index, value) {
            var v=$(value).attr('value');
            if($(value).attr('selected')==="selected") {
                options.selected.push(v);
            }
            values.push(v);
            optionText.push($(value).text());
        }
        );
        var id=select.attr('id');
        var warpId=id + '-selectDialogWarp';
        var contentId=id + '-content';
        var dialogId=id + '-dialogId';
        var open_dialog=function() {
            var d=dialog( {
                okValue: options.okValue,
                cancelValue: options.cancelValue,
                title: options.title, width: options.width, height: options.height, lock: true, content: creat_content(optionText), ok: function() {
                    $(document).unbind('keydown');
                    creat_warp_content();
                    //点保存以后，设置select值
                    select.find('option').each(function(index, value) {
                        var option=$(value), v=option.attr('value');
                        if($.inArray(v, options.selected)===-1) {
                            option.removeAttr('selected');
                        }
                        else {
                            option.attr('selected', 'selected')
                        }
                    }
                    );
                    select.trigger('change');
                }
                , onshow: function() {
                    //对话框弹出时执行的函数
                    $(".ui-dialog-body").css('vertical-align', 'top');
                    $(".ui-dialog-body").css('text-align', 'left');
                    $(".ui-dialog-content .btn-toolbar .btn").click(function() {
                        //绑定按钮点击事件
                        btnClick(this);
                    }
                    );
                    $(".ui-dialog-content").find('button[data-id="check_all"]').click(function() {
                        check_all();
                    }
                    );
                    $(".ui-dialog-content").find('button[data-id="inverse"]').click(function() {
                        inverse();
                    }
                    );
                    $(document).keyup(function(event) {
                        if(event.keyCode==27) {
                            d.close();
                        }
                    }
                    );
                    //过滤
                    $('input[data-id="keyword"]').bind('keyup', filter);
                    //如果是单选，隐藏全选和反选
                    if(module==='single') {
                        $('#'+dialogId).find('div[data-id="btn-group"]').hide();
                    }
                }
                , okVal: options.okVal, cancel: function() {
                    $(".ui-dialog-body").css('vertical-align', 'middle');
                    $(".ui-dialog-body").css('text-align', 'center');
                    $(".ui-dialog-close").trigger('mouseout');
                    $(document).unbind('keydown');
                }
                , cancelVal: options.cancelVal
            });
            d.showModal();
        }
        var check_all=function() {
            $(".ui-dialog-content .btn-toolbar .btn").each(function() {
                var v=values[parseInt($(this).attr('data-index'))];
                if(v && !$(this).is(':hidden')) {
                    add(v);
                    $(this).addClass('btn-primary');
                }
            }
            );
        }
        var inverse=function() {
            $(".ui-dialog-content .btn-toolbar .btn").each(function() {
                if(!$(this).is(':hidden'))$(this).trigger('click');
            }
            );
            //反选后执行一次过滤操作
            $('input[data-id="keyword"]').trigger('keyup');
        }
        var btnClick=function (btn) {
            var btn=$(btn);
            var value=values[parseInt(btn.attr('data-index'))];
            if(btn.hasClass('btn-primary')) {
                btn.removeClass('btn-primary');
                remove(value);
            }
            else {
                //单选模式下，取消其他选中的
                if(module==='single') {
                    $('#'+ contentId + ' .btn-primary').each(function(index, v) {
                        $(v).trigger('click');
                    }
                    );
                }
                btn.addClass('btn-primary');
                add(value);
            }
        }
        var add=function(v) {
            if($.inArray(v, options.selected)===-1) {
                options.selected.push(v);
            }
        }
        var remove=function(v) {
            var key=$.inArray(v, options.selected);
            options.selected.splice(key, 1);
        }
        var creat_warp_content=function() {
            var content='<div class="btn-toolbar">';
            if(options.selected.length===0 && options.emptyText) {
                content +='<span style="color:#999999;">'+ options.emptyText +'</span>'
            }
            else {
                $.each(options.selected, function(index, v) {
                    var key=$.inArray(v, values);
                    var value=optionText[key];
                    if(value) {
                        content +='<div class="btn-group"><a class="btn btn-primary btn-xs" data="'+v+'">'+ value +'</a><button type="button" class="u-btn-close btn btn-primary btn-xs dropdown-toggle"><span></span></button></div>';
                    }
                }
                );
            }
            content +='</div>';
            $('#'+warpId).html(content);
            $('.u-btn-close').on('click',function (){
                var closeIndex=optionText.indexOf($(this).siblings('a').text());
                $(this).parent().remove();
                select.find('option').eq(closeIndex).removeAttr('selected');
                var _closeIndex=options.selected.indexOf(select.find('option').eq(closeIndex).val());
                options.selected.splice(_closeIndex,1);
                return false;
            })
        }
        ;
        var creat_content=function(data) {
            var toolbar='<div class="container-fluid"><div class="row" style="height:44px;" id="'+ dialogId +'">'+ '<div class="col-xs-8" style="padding:0;">'+ '<div class="btn-group btn-group-sm" data-id="btn-group" style="margin-bottom: 10px;">'+ '<button class="btn btn-primary" data-id="check_all">全选</button>'+ '<button class="btn btn-success" data-id="inverse">反选</button>'+ '</div>'+ '</div>'+ '<div class="col-xs-4" style="padding:0;">'+ '<input type="text" data-id="keyword" placeholder="搜索..." style="float: right;margin-top:  7px;margin-right: 6px;width:200px;">'+ '</div>'+ '</div></div>';
            var content=toolbar+'<div class="btn-toolbar" id='+ contentId +' style="width:700px;">';
            $.each(data, function(index, v) {
                var default_class='btn';
                if($.inArray(values[index], options.selected) > -1) {
                    default_class +=' btn-primary'
                }
                content +='<div class="btn-group btn-group-sm"><button class="'+ default_class +'" data-index="'+index+'" data-text="'+ v.toLowerCase() +'">'+ v +'</button></div>';
            }
            );
            content +='</div>';
            return content;
        }
        var filter=function(event) {
            var input=$(event.target), key=input.val().toLowerCase(), content=$('#'+contentId);
            if(key=='') {
                content.find('button').show();
            }
            else {
                var buttons=content.find('button:not([data-text*="'+ key +'"]):not(.btn-primary)');
                content.find('button').show(); //避免输入中文时的bug
                buttons.hide();
            }
        }
        this.each(function() {
            $(this).hide();
            if($('#'+warpId).length>0) {
                //如果存在，清空内容
                $('#'+warpId).html('').off('click');
            }
            else {
                $(this).after('<div class="form-control" id="'+warpId+'" style="min-height:37px;height:auto;cursor: pointer;"></div>');
            }
            creat_warp_content();
            $('#'+warpId).click(function() {
                open_dialog(options);
            }
            );
        }
        );
        return this;
    }
})(jQuery);
```

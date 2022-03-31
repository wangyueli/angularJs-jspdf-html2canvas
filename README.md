# angularJs-jspdf-html2canvas
angularJs 将html网页转换为pdf文件并发送给后端

#资源 
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script src="//static.yuncaitong.cn/asset/html2canvas.min.js"></script>
<script src="https://cdn.bootcss.com/jspdf/1.5.3/jspdf.debug.js"></script>





#js代码









     $scope.loadPdf = function () {
        let printEle = document.getElementById('print-pdf')
        html2canvas(printEle, {
            onrendered:function(canvas) {
                var contentWidth = canvas.width;
                var contentHeight = canvas.height;

                //一页pdf显示html页面生成的canvas高度;
                var pageHeight = contentWidth / 595.28 * 841.89;
                //未生成pdf的html页面高度
                var leftHeight = contentHeight;
                //pdf页面偏移
                var position = 0;
                //a4纸的尺寸[595.28,841.89]，html页面生成的canvas在pdf中图片的宽高
                var imgWidth = 555.28;
                var imgHeight = 555.28/contentWidth * contentHeight;

                var pageData = canvas.toDataURL('image/jpeg', 1.0);

                var pdf = new jsPDF('', 'pt', 'a4');
                //有两个高度需要区分，一个是html页面的实际高度，和生成pdf的页面高度(841.89)
                //当内容未超过pdf一页显示的范围，无需分页
                if (leftHeight < pageHeight) {
                    pdf.addImage(pageData, 'JPEG', 20, 0, imgWidth, imgHeight );
                } else {
                    while(leftHeight > 0) {
                        pdf.addImage(pageData, 'JPEG', 20, position, imgWidth, imgHeight)
                        leftHeight -= pageHeight;
                        position -= 841.89;
                        //避免添加空白页
                        if(leftHeight > 0) {
                            pdf.addPage();
                        }
                    }
                }
                
                // 将pdf输入为base格式的字符串
                var buffer = pdf.output("datauristring")
                // 将base64格式的字符串转换为file文件
                var myfile = $scope.dataURLtoFile(buffer, '生成的pdf文件.pdf')
                var formdata = new FormData()
                formdata.append('file', myfile)
                let url = 'https://file.yuncaitong.cn'
                // 之后ajax传递数据
                var  xhr = new XMLHttpRequest()
                xhr.open('POST', url, false)
                xhr.send(formdata)
                console.log(xhr.responseText); 

            }
        })
    }

    //将base64转换为文件对象
    $scope.dataURLtoFile = function (dataurl, filename) {
        var arr = dataurl.split(',');
        var mime = arr[0].match(/:(.*?);/)[1];
        var bstr = atob(arr[1]);
        var n = bstr.length; 
        var u8arr = new Uint8Array(n);
        while(n--){
            u8arr[n] = bstr.charCodeAt(n);
        }
        //转换成file对象
        return new File([u8arr], filename, {type:mime});
        //转换成成blob对象
        // return new Blob([u8arr],{type:mime});
    }

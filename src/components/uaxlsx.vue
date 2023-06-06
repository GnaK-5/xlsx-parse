<script setup>
import * as XLSX from 'xlsx';

const resObj = {};
let andTotal = 0;
let iOSTotal = 0;
let disAndTotal = 0;
let disIOSTotal = 0;

const handleFileUpload = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();

    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const sheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[sheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

        getBrowserVersion(jsonData);

        console.log({ andTotal, iOSTotal, disAndTotal, disIOSTotal });
    };

    reader.readAsArrayBuffer(file);
};

const getBrowserVersion = (arr) => {
    arr.forEach((item) => {
        const ua = item[0];
        if (!ua) return;
        const uv = item[1];

        const AndroidStartString = 'Chrome/';
        const andIndex = ua.indexOf(AndroidStartString);
        if (andIndex !== -1) {
            andTotal += uv;
            const substringStartIndex = andIndex + AndroidStartString.length;
            const res = ua.substr(substringStartIndex, 3).split('.')[0];
            const resStr = 'Chrome ' + res;
            if (resObj[resStr]) {
                resObj[resStr] += uv;
            } else {
                resObj[resStr] = uv;
            }
            if (res < 51) disAndTotal += uv;
            return;
        }

        const iOSStartString = 'OS ';
        const iosIndex = ua.indexOf(iOSStartString);
        if (iosIndex !== -1) {
            iOSTotal += uv;
            const substringStartIndex = iosIndex + iOSStartString.length;
            const res = ua.substr(substringStartIndex, 3).split('_')[0];
            const resStr = 'iOS' + res;
            if (resObj[resStr]) {
                resObj[resStr] += uv;
            } else {
                resObj[resStr] = uv;
            }
            console.log(res);
            if (res < 10) disIOSTotal += uv;
            return;
        }

        // console.log(ua);
    });
};

const handleXlsx = () => {
    const workbook = XLSX.utils.book_new();
    const xlsxArr = [['浏览器版本', 'uv']];
    const resKeyArr = Object.keys(resObj);
    resKeyArr.forEach((key) => {
        const curArr = [key];
        curArr.push(resObj[key]);
        xlsxArr.push(curArr);
    });
    xlsxArr.push(['Chrome总数', andTotal]);
    xlsxArr.push(['iOS总数', iOSTotal]);
    console.log(xlsxArr);

    const worksheet = XLSX.utils.aoa_to_sheet(xlsxArr);
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

    const wbout = XLSX.write(workbook, { type: 'array', bookType: 'xlsx' });

    const blob = new Blob([wbout], { type: 'application/octet-stream' });

    const url = URL.createObjectURL(blob);

    const link = document.createElement('a');
    link.href = url;
    link.download = 'output.xlsx';

    link.click();

    URL.revokeObjectURL(url);
};
</script>

<template>
    <div class="xlsx-box">
        <input type="file" @change="handleFileUpload" />
        <button @click="handleXlsx">获取xlsx文件</button>
    </div>
</template>

<style scoped>
.xlsx-box {
    margin-top: 300px;
    display: flex;
    justify-content: center;
}
</style>

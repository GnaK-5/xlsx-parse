<script setup>
import * as XLSX from 'xlsx';

const resObj = {};

const handleFileUpload = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();

    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const sheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[sheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

        // console.log(jsonData);
        getProjectData(jsonData);
        // console.log(resObj);
    };

    reader.readAsArrayBuffer(file);
};

const getProjectData = (arr) => {
    arr.forEach((item) => {
        const url = item[1];
        const pv = item[3];
        const startString = '-assist-';
        if (!url) return;
        const startIndex = url.indexOf(startString);

        if (startIndex !== -1) {
            const substringStartIndex = startIndex + startString.length;
            const substringEndIndex = url.indexOf('/', substringStartIndex);
            if (substringEndIndex !== -1) {
                const result = url.substring(
                    substringStartIndex,
                    substringEndIndex
                );
                if (resObj[result]) {
                    resObj[result] += pv;
                } else {
                    resObj[result] = pv;
                }
            } else {
                console.log('/');
            }
        }
    });
};

const handleXlsx = () => {
    const workbook = XLSX.utils.book_new();
    const xlsxArr = [['项目名称', 'pv']];
    const resKeyArr = Object.keys(resObj);
    resKeyArr.forEach((key) => {
        const curArr = [key];
        curArr.push(resObj[key]);
        xlsxArr.push(curArr);
    });
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

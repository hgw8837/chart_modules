<template>
    <div>
        <input type="file" id="input-file">
        <button :onclick="fileReadAPI">파일 데이터 로드</button>
        <input v-model="datasetName" placeholder="테이블 명을 입력해 주세요">
        <button :onclick="datasetInsert">저장</button>
    </div>
    <div>
        <table class="file-data-list">
            <thead>
                <td v-for="(item, key, idx) in fileReadData['columnDatas']" :key="key">{{ item['originColumnName'] }}</td>
            </thead>
            <tbody>
                <tr v-for="(items, keys, idxs) in fileReadData['rowData']" :key="keys">
                    <td v-for="(item, key, idx) in fileReadData['rowData'][keys]" :key="key">{{ item }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data () {
        return {
            // 선택한 파일 원본 정보
            fileSelectDefaultData: [],
            // 서버로 보낼 선택한 파일 데이터
            fileSelectData : {
                // 원본 파일 명
                fileName: null,
                // 원본 파일 확장자
                fileType: null,
            },

            // 읽은 파일 정보
            fileReadData: [],
            // 데이터셋 명
            datasetName: null,
        }
    },
    methods: {

        // 파일 데이터(xlsx) 읽기 API
        fileReadAPI: function() {
            const vm = this;
            this.fileSelectDefaultData = [];

            console.log("document.getElementById('input-file').files : ", document.getElementById("input-file").files);

            const files = document.getElementById("input-file").files;
            this.fileSelectDefaultData = files;
            // 파일 명을 구분자로 구분
            let fileSplit = files[0]['name'].split('.', 2);
            // 원본 파일 이름
            this.fileName = fileSplit[0];
            // 원본 파일 타입
            this.fileType = fileSplit[1];

            const formData = new FormData();
            for (let i = 0; i < files.length; i++) {
                formData.append('files', files[i]);
            }

            //fetch post
            // fetch("http://165.229.169.112:8000/file/datasetFileRead.ajax", {
            fetch("http://165.229.169.139:8060/file/datasetFileRead.ajax", {
                method: "POST",
                /* headers: {
                    'Content-Type': 'application/json; charset=UTF-8'
                }, */
                body: formData
            }).then((response) => response.json()).then((data) => {
                console.log('fileReadAPI response : ', data);

                // 선택한 파일명과 읽은 파일명이 같을 경우
                if(vm.fileName == data[`${this.fileName}`]['title']){
                    // 컬럼 데이터
                    vm.fileReadData['columnDatas'] = data[`${this.fileName}`]['columnDatas'];
                    // row 데이터
                    vm.fileReadData['rowData'] = data[`${this.fileName}`]['rowData'];
                }
            }).catch((error) => {
                console.log('fileReadAPI error : ', error);
            });
        },

        // 데이터셋 등록
        datasetInsert: function() {
            const vm = this;

            axios({
                url: '/dataset/datasetInsert.json',
				method: 'post',
				headers: {
					'Content-Type': "application/json; charset=UTF-8",
				},
				data: {
                    'fileOrigin' : vm.fileSelectDefaultData[0]['name'], // 원본 파일 명
                    'fileSaveName' : vm.datasetName, // 저장시 파일 명 (현재는 임시로 테이블 명 사용)
                    'fileSavedirection' : null, // 원본 파일 저장 경로
                    'fileSize' : vm.fileSelectDefaultData[0]['size'], // 원본 파일 크기
                    'fileType' : vm.fileType, // 원본 파일 타입
                    'tableName' : vm.datasetName, // 테이블 명
                    'columnDatas' : vm.fileReadData['columnDatas'], // 컬럼 데이터
                    'rowData' : vm.fileReadData['rowData'] // row 데이터
                }
			}).then(function (response) {
				console.log("datasetInsert - response : ", response.data);
                let result = response.data;
                if (result > 0) {
                    alert('데이터셋 등록에 성공하였습니다.');
                    
                    // 등록 성공 시 선택한 파일, 입력받은 테이블 명 등 모두 초기화
                    vm.fileSelectDefaultData = [];
                    vm.fileSelectData = {};
                    vm.fileReadData = [];
                    vm.datasetName = null;

                } else {
                    alert('데이터셋 등록에 실패하였습니다.');
                }
			}).catch(function (error) {
				console.log("datasetInsert - error : ", error);
				alert('데이터셋 등록 오류');
			});
        },
    },
    watch: {

    },
    computed: {

    },
    components: {

    },
    mounted() {
        console.log('main mounted');
    }
}
</script>

<style scoped>
#chartdiv {
  width: 100%;
  height: 500px;
}
.file-data-list > thead > td {
    background-color: beige;
}
.file-data-list > tbody > tr > td {
    border: 1px solid #444444;
}
</style>
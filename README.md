### DevExtreme UA localization setup
```javascript
fetch('https://raw.githubusercontent.com/SadJuly/external-sources/main/devextreme_ua.json')
.then(res => res.json())
.then(data => {
	DevExpress.localization.loadMessages(data)
	DevExpress.localization.locale('ua')
})
```


### Custom page setup

In MainPage.js

```javascript
scripts: [
	'https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/js/select2.full.js'
],
styles: [
	'https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/css/select2.min.css',
	'https://cdn.jsdelivr.net/npm/select2-bootstrap-theme@0.1.0-beta.10/dist/select2-bootstrap.min.css'
],
init: function () {

	const loadCDN = () => {	
		let scriptsPromises = this.scripts.map(src => {
			return new Promise(function (resolve, reject) {
				let script = document.createElement('script');
				script.src = src;

				script.onload = () => resolve(script);
				script.onerror = () => reject(new Error(`Ошибка загрузки скрипта ${src}`));

				document.head.append(script);
			});
		})

		let stylesPromises = this.styles.map(href => {
			return new Promise(function (resolve, reject) {
				let script = document.createElement('link');
				script.href = href;
				script.rel = 'stylesheet';

				script.onload = () => resolve(script);
				script.onerror = () => reject(new Error(`Ошибка загрузки скрипта ${href}`));

				document.head.append(script);
			});
		})

		return Promise.all([...scriptsPromises, ...stylesPromises])

	}

	window.helperFunctions = {
		loadCDN: loadCDN,
		normalizeData: this.normalizeData,
		promiseQuery: this.promiseQuery

	}
},
promiseQuery(queryObject) {
	return new Promise((res, rej) => {

		this.queryExecutor(queryObject, data => {
			res(data)
		})

	})
},
normalizeData: function (data, byName = true, oneRow = false) {

	let result = []
	data.rows.forEach((row, rowIndex) => {
		let obj = {}
		data.columns.forEach((col, colIndex) => {
			obj[byName ? col.name : col.code] = data.rows[rowIndex].values[colIndex]
		})
		result.push(obj)
	})

	return oneRow ? result[0] : result;
}

```


In CustomWidget-0.js

```javascript
afterViewInit: function () {

	const {loadCDN, normalizeData, promiseQuery} = window.helperFunctions
	this.normalizeData = normalizeData
	this.promiseQuery = promiseQuery
	loadCDN()
	.then(cdns => {
		//do stuff after cdns loaded on page

	})
	.catch(err => {
		console.log(err)
	})
},
```




select2 ajax optimized select

```javascript
createSelect2Ajax({element, valueAlias, select2Config, queryConfig}){
	let {id, text} = valueAlias
	return $(element).select2({
		allowClear: true,
		language: 'ru',
		placeholder: '',
		selectionCssClass: 'form-control',
		theme: 'bootstrap',
		ajax: {
			method: 'POST',
			url: '/api/ds/utility/query/getpagevalues',
			headers: {
				"accept": "application/json, text/plain, */*",
				"authorization": "Bearer " + localStorage.getItem('X-Auth-Token'),
				"content-type": "application/json",
			},
			"mode": "cors",
			"credentials": "include",
			data: (params) => {

				let getPageValuesParams = JSON.stringify(queryConfig(params))

				return getPageValuesParams

			} ,
			processResults: (data, params) => {
				let normalData = this.normalizeData(data, false)
				let mappedData = normalData.map(el => ({
					id: el[id],
					text: el[text]
				}))

				return {
					results: mappedData,
					pagination: {
						more: mappedData.length == 15
					}
				};


			},
		},
		...select2Config
	})
}
```

Using example

```javascript
this.createSelect2Ajax({
	element: '#search-admin-unit',
	valueAlias: {
		id: "Code",
		text: "Name"
	},
	queryConfig: (params) => ({
		queryCode: 'List_AdministrativeUnits',
		filterColumns: [
			{
				key: "Name",
				value: {
					operation: 6,
					not: false,
					values: [
						params.term || ''
					]
				}
			}
		],
		limit: -1,
		parameterValues: [],
		pageNumber: params.page || 1,
		sortColumns: [
			{
				key: "Name",
				value: 0
			}
		]
	}),
	select2Config: {}
})
.on('select2:select', (e) => {
	// some logic
})
```

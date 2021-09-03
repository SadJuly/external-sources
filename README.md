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

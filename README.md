### DevExtreme UA localization setup
```javascript
fetch('https://raw.githubusercontent.com/SadJuly/external-sources/main/devextreme_ua.json')
.then(res => res.json())
.then(data => {
	DevExpress.localization.loadMessages(data)
	DevExpress.localization.locale('ua')
})
```

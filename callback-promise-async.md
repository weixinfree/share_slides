## 异步编程的进化

```
callback -> Promise -> Async/Await
```

[slide]

## callback

```js
httpGet({uid = '1234567890'}, onSuccess = (response) => {
	// 
}, error = (code, msg) => {
	// 
})
```

[slide]

## Promise

```js
scaleImage(this.props.portrait, { width: 108, height: 108 }).then(
    scaleUrl => {
        this.setState({
            imgUrl: scaleUrl
        })
    }
);
```

[slide]

## Async/Await
```js
/**
 	图片缩放服务: 
	quality: 图像质量，默认为80
	origin: 需要原图还是缩放后的图，1 原图 0 缩放后的图，默认采用缩放后的图
	format: 图片格式, webp = 1, png = 0。 默认采用png
 */
async function scaleImage(baseUrl, {
	width,
	height,
	quality = 80,
	origin = 0,
	format = 0
}) {
	if (!baseUrl.startsWith('http://')&& !baseUrl.startsWith('https://')) {
		let host = await IKServiceInfo.getUrl('IMAGE')
		baseUrl = host + baseUrl
	}
	const url  = await IKServiceInfo.getUrl('IMAGE_SCALE')
	let encodedBaseUrl = encodeURIComponent(baseUrl)
	return `${url}${encodedBaseUrl}&w=${width}&h=${height}&s=${quality}&c=0&o=${origin}&t=${format}`
}

export default scaleImage;
```
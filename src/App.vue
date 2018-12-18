<template>
  <div id="app">
  	<hello-world></hello-world>
    <router-view/>
  </div>
</template>

<script>
	document.addEventListener('DOMContentLoaded', () => {
		const html = document.querySelector('html');
		let fontSize = window.innerWidth / 10;
		fontSize = fontSize > 50 ? 50 : fontSize;
		html.style.fontSize = fontSize + 'px';
	})

	export default {
		components: {
			HelloWorld,
		}
	}
</script>

<style>

</style>

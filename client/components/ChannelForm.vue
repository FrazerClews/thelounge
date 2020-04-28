<!-- Check max length for topic -->
<template>
	<div id="channel" class="window" role="tabpanel" aria-label="Channel">
		<div class="header">
			<SidebarToggle />
		</div>
		<form class="container" method="post" action="" @submit.prevent="onSubmit">
			<h1>Hello World</h1>
		</form>
	</div>
</template>

<!-- bring back revealpassword -->

<script>
import SidebarToggle from "./SidebarToggle.vue";

export default {
	name: "ChannelForm",
	components: {
		SidebarToggle,
	},
};
</script>

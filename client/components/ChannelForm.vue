<!-- Check max length for topic -->
<template>
	<div id="connect" class="window" role="tabpanel" aria-label="Connect">
		<div class="header">
			<SidebarToggle />
		</div>
		<form class="container" method="post" action="" @submit.prevent="onSubmit">
			<h1 class="title">
				<template v-if="defaults.id">
					<input v-model="defaults.id" type="hidden" name="id" />
					Edit {{ defaults.name }}
				</template>
				<template v-else>
					Connect
					<template v-if="config.lockChannel">to {{ defaults.name }}</template>
				</template>
			</h1>
			<template v-if="!config.lockChannel">
				<h2>Channel settings</h2>
				<div class="connect-row">
					<label for="connect:name">Name</label>
					<input
						id="connect:name"
						v-model="defaults.name"
						class="input"
						name="name"
						maxlength="100"
					/>
				</div>
				<div class="connect-row">
					<label for="connect:password">Password</label>
					<RevealPassword
						v-slot:default="slotProps"
						class="input-wrap password-container"
					>
						<input
							id="connect:password"
							v-model="defaults.password"
							class="input"
							:type="slotProps.isVisible ? 'text' : 'password'"
							placeholder="Server password (optional)"
							name="password"
							maxlength="300"
						/>
					</RevealPassword>
				</div>
				<div class="connect-row">
					<label for="connect:topic">Topic</label>
					<input
						id="connect:topic"
						v-model="defaults.topic"
						class="input"
						name="name"
						maxlength="512"
					/>
				</div>
				<div class="connect-row">
					<label></label>
					<div class="input-wrap">
						<label class="mute">
							<input v-model="defaults.mute" type="checkbox" name="mute" />
							Mute Channel
						</label>
					</div>
				</div>
			</template>
			<h2>User preferences</h2>
			<template v-if="defaults.id && !$store.state.serverConfiguration.public">
				<div class="connect-row">
					<label for="connect:commands">
						Commands
						<span
							class="tooltipped tooltipped-ne tooltipped-no-delay"
							aria-label="One /command per line.
Each command will be executed in
the server tab on new connection"
						>
							<button class="extra-help" />
						</span>
					</label>
					<textarea
						id="connect:commands"
						ref="commandsInput"
						:value="defaults.commands ? defaults.commands.join('\n') : ''"
						class="input"
						name="commands"
						@input="resizeCommandsInput"
					/>
				</div>
			</template>
			<template v-else-if="!defaults.id">
				<div class="connect-row">
					<label for="connect:channels">Channels</label>
					<input
						id="connect:channels"
						v-model="defaults.join"
						class="input"
						name="join"
					/>
				</div>
			</template>

			<template v-if="$store.state.serverConfiguration.public">
				<template v-if="config.lockChannel">
					<div class="connect-row">
						<label></label>
						<div class="input-wrap">
							<label class="tls">
								<input v-model="displayPasswordField" type="checkbox" />
								I have a password
							</label>
						</div>
					</div>
					<div v-if="displayPasswordField" class="connect-row">
						<label for="connect:password">Password</label>
						<RevealPassword
							v-slot:default="slotProps"
							class="input-wrap password-container"
						>
							<input
								id="connect:password"
								ref="publicPassword"
								v-model="defaults.password"
								class="input"
								:type="slotProps.isVisible ? 'text' : 'password'"
								placeholder="Server password (optional)"
								name="password"
								maxlength="300"
							/>
						</RevealPassword>
					</div>
				</template>
			</template>

			<div>
				<button type="submit" class="btn" :disabled="disabled ? true : false">
					<template v-if="defaults.id">Save channel</template>
					<template v-else>Connect</template>
				</button>
			</div>
		</form>
	</div>
</template>

<style>
#connect .connect-auth {
	display: block;
	margin-bottom: 10px;
}

#connect .connect-auth .opt {
	display: block;
	width: 100%;
}

#connect .connect-auth input {
	margin: 3px 10px 0 0;
}

#connect .connect-sasl-external {
	padding: 10px;
	border-radius: 2px;
	background-color: #d9edf7;
	color: #31708f;
}

#connect .connect-sasl-external pre {
	margin: 0;
	user-select: text;
}
</style>

<!-- bring back revealpassword -->

<script>
import SidebarToggle from "./SidebarToggle.vue";

export default {
	name: "ChannelForm",
	components: {
		SidebarToggle,
	},
	props: {
		handleSubmit: Function,
		defaults: Object,
		disabled: Boolean,
	},
	data() {
		return {
			config: this.$store.state.serverConfiguration,
			previousUsername: this.defaults.username,
			displayPasswordField: false,
		};
	},
	watch: {
		displayPasswordField(value) {
			if (value) {
				this.$nextTick(() => this.$refs.publicPassword.focus());
			}
		},
		"defaults.commands"() {
			this.$nextTick(this.resizeCommandsInput);
		},
	},
	methods: {
		onSubmit(event) {
			const formData = new FormData(event.target);
			const data = {};

			for (const item of formData.entries()) {
				data[item[0]] = item[1];
			}

			this.handleSubmit(data);
		},
		resizeCommandsInput() {
			if (!this.$refs.commandsInput) {
				return;
			}

			// Reset height first so it can down size
			this.$refs.commandsInput.style.height = "";

			// 2 pixels to account for the border
			this.$refs.commandsInput.style.height =
				Math.ceil(this.$refs.commandsInput.scrollHeight + 2) + "px";
		},
	},
};
</script>

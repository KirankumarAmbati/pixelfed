<template>
	<div>
		<!-- <div class="bg-white py-4">
			<div class="container">
				<div class="d-flex justify-content-between align-items-center">
					<div></div>
					<a href="/account/activity" class="cursor-pointer font-weight-bold text-primary">Notifications</a>
					<a href="/account/direct" class="cursor-pointer font-weight-bold text-dark">Direct Messages</a>
					<a href="/account/following" class="cursor-pointer font-weight-bold text-dark">Following</a>
					<div></div>
				</div>
			</div>
		</div> -->
		<div class="container">
			<div class="row my-5">
				<div class="col-12 col-md-8 offset-md-2">
					<div v-if="notifications.length > 0">				
						<div class="media mb-3 align-items-center px-3 border-bottom pb-3" v-for="(n, index) in notifications" :key="index">
							<img class="mr-2 rounded-circle" style="border:1px solid #ccc" :src="n.account.avatar" alt="" width="32px" height="32px">
							<div class="media-body font-weight-light">
								<div v-if="n.type == 'favourite'">
									<p class="my-0">
										<a :href="n.account.url" class="font-weight-bold text-dark word-break" data-placement="bottom" data-toggle="tooltip" :title="n.account.username">{{truncate(n.account.username)}}</a> liked your <a class="font-weight-bold" v-bind:href="n.status.url">post</a>.
									</p>
								</div>
								<div v-else-if="n.type == 'comment'">
									<p class="my-0">
										<a :href="n.account.url" class="font-weight-bold text-dark word-break" data-placement="bottom" data-toggle="tooltip" :title="n.account.username">{{truncate(n.account.username)}}</a> commented on your <a class="font-weight-bold" v-bind:href="n.status.url">post</a>.
									</p>
								</div>
								<div v-else-if="n.type == 'mention'">
									<p class="my-0">
										<a :href="n.account.url" class="font-weight-bold text-dark word-break" data-placement="bottom" data-toggle="tooltip" :title="n.account.username">{{truncate(n.account.username)}}</a> <a class="font-weight-bold" v-bind:href="mentionUrl(n.status)">mentioned</a> you.
									</p>
								</div>
								<div v-else-if="n.type == 'follow'">
									<p class="my-0">
										<a :href="n.account.url" class="font-weight-bold text-dark word-break" data-placement="bottom" data-toggle="tooltip" :title="n.account.username">{{truncate(n.account.username)}}</a> followed you.
									</p>
								</div>
								<div v-else-if="n.type == 'share'">
									<p class="my-0">
										<a :href="n.account.url" class="font-weight-bold text-dark word-break" data-placement="bottom" data-toggle="tooltip" :title="n.account.username">{{truncate(n.account.username)}}</a> shared your <a class="font-weight-bold" v-bind:href="n.status.reblog.url">post</a>.
									</p>
								</div>
								<div class="align-items-center">
									<span class="small text-muted" data-toggle="tooltip" data-placement="bottom" :title="n.created_at">{{timeAgo(n.created_at)}}</span>
								</div>
							</div>
							<div>
								<div v-if="n.status && n.status && n.status.media_attachments && n.status.media_attachments.length">
									<a :href="n.status.url">
										<img :src="n.status.media_attachments[0].preview_url" width="32px" height="32px">
									</a>
								</div>
								<div v-else-if="n.status && n.status.parent && n.status.parent.media_attachments && n.status.parent.media_attachments.length">
									<a :href="n.status.parent.url">
										<img :src="n.status.parent.media_attachments[0].preview_url" width="32px" height="32px">
									</a>
								</div>
								<!-- <div v-else-if="n.status && n.status.parent && n.status.parent.media_attachments && n.status.parent.media_attachments.length">
									<a :href="n.status.parent.url">
										<img :src="n.status.parent.media_attachments[0].preview_url" width="32px" height="32px">
									</a>
								</div> -->

								<!-- <div v-else-if="n.type == 'follow' && n.relationship.following == false">
									<a href="#" class="btn btn-primary py-0 font-weight-bold" @click.prevent="followProfile(n);">
										Follow
									</a>
								</div> -->
								
								<!-- <div v-else-if="n.status && n.status.parent && !n.status.parent.media_attachments && n.type == 'like' && n.relationship.following == false">
									<a href="#" class="btn btn-primary py-0 font-weight-bold">
										Follow
									</a>
								</div> -->
								<div v-else>
									<a class="btn btn-outline-primary py-0 font-weight-bold" :href="viewContext(n)">View</a>
								</div>
							</div>
						</div>
						<div>
							<infinite-loading @infinite="infiniteNotifications">
								<div slot="no-results" class="font-weight-bold"></div>
								<div slot="no-more" class="font-weight-bold"></div>
							</infinite-loading>
						</div>
					</div>
					<div v-if="notifications.length == 0" class="text-lighter text-center py-3">
						<p class="mb-0"><i class="fas fa-inbox fa-3x"></i></p>
						<p class="mb-0 small font-weight-bold">No new notifications!</p>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script type="text/javascript">
export default {
	data() {
		return {
			notifications: {},
			notificationCursor: 2,
			notificationMaxId: 0,
		};
	},

	mounted() {
		this.fetchNotifications();
	},

	updated() {
		$('[data-toggle="tooltip"]').tooltip()
	},

	methods: {
		fetchNotifications() {
			axios.get('/api/v1/notifications')
			.then(res => {
				let data = res.data.filter(n => {
					if(n.type == 'share' && !status) {
						return false;
					}
					return true;
				});
				let ids = res.data.map(n => n.id);
				this.notificationMaxId = Math.max(...ids);
				this.notifications = data;
				$('.notification-card .loader').addClass('d-none');
				$('.notification-card .contents').removeClass('d-none');
			});
		},

		infiniteNotifications($state) {
			if(this.notificationCursor > 10) {
				$state.complete();
				return;
			}
			axios.get('/api/v1/notifications', {
				params: {
					page: this.notificationCursor
				}
			}).then(res => {
				if(res.data.length) {
					let data = res.data.filter(n => {
						if(n.type == 'share' && !status) {
							return false;
						}
						if(_.find(this.notifications, {id: n.id})) {
							return false;
						}
						return true;
					});
					this.notifications.push(...data);
					this.notificationCursor++;
					$state.loaded();
				} else {
					$state.complete();
				}
			});
		},
		
		truncate(text) {
			if(text.length <= 15) {
				return text;
			}

			return text.slice(0,15) + '...'
		},

		timeAgo(ts) {
			let date = Date.parse(ts);
			let seconds = Math.floor((new Date() - date) / 1000);
			let interval = Math.floor(seconds / 31536000);
			if (interval >= 1) {
				return interval + "y";
			}
			interval = Math.floor(seconds / 604800);
			if (interval >= 1) {
				return interval + "w";
			}
			interval = Math.floor(seconds / 86400);
			if (interval >= 1) {
				return interval + "d";
			}
			interval = Math.floor(seconds / 3600);
			if (interval >= 1) {
				return interval + "h";
			}
			interval = Math.floor(seconds / 60);
			if (interval >= 1) {
				return interval + "m";
			}
			return Math.floor(seconds) + "s";
		},

		mentionUrl(status) {
			let username = status.account.username;
			let id = status.id;
			return '/p/' + username + '/' + id;
		},

		followProfile(n) {
			let self = this;
			let id = n.account.id; 
			axios.post('/i/follow', {
					item: id
			}).then(res => {
				self.notifications.map(notification => {
					if(notification.account.id === id) {
						notification.relationship.following = true;
					}
				});
			}).catch(err => {
				if(err.response.data.message) {
					swal('Error', err.response.data.message, 'error');
				}
			});
		},

		viewContext(n) {
			switch(n.type) {
				case 'follow':
					return n.account.url;
				break;
				case 'mention':
				case 'like':
				case 'favourite':
				case 'comment':
					return n.status.url;
				break;
			}
			return '/';
		},
	}
}
</script>
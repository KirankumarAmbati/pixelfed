<template>
	<div>
		<div v-if="loaded" class="container">
			<div class="profile-header row my-5">
				<div class="col-12 col-md-3">
					<div class="profile-avatar">
						<div class="bg-pixelfed mb-3 d-flex align-items-center justify-content-center display-4 font-weight-bold text-white" style="width: 172px; height: 172px; border-radius: 100%">#</div>
					</div>
				</div>
				<div class="col-12 col-md-9 d-flex align-items-center">
					<div class="profile-details">
						<div class="username-bar pb-2">
							<p class="tag-header mb-0">#{{hashtag}}</p>
							<p class="lead"><span class="font-weight-bold">{{tags.length ? hashtagCount : '0'}}</span> posts</p>
							<p v-if="authenticated && tags.length" class="pt-3">
								<button v-if="!following" type="button" class="btn btn-primary font-weight-bold py-1 px-5" @click="followHashtag">
									Follow
								</button>
								<button v-else type="button" class="btn btn-outline-secondary font-weight-bold py-1 px-5" @click="unfollowHashtag">
									Unfollow
								</button>
							</p>
						</div>
					</div>
				</div>
			</div>
			<div v-if="tags.length" class="tag-timeline">
				<p v-if="top.length" class="font-weight-bold text-muted mb-0">Top Posts</p>
				<div class="row pb-5">
					<div v-for="(tag, index) in top" :key="index" class="col-4 p-0 p-sm-2 p-md-3 hashtag-post-square">
						<a class="card info-overlay card-md-border-0" :href="tag.status.url">
							<div :class="[tag.status.filter ? 'square ' + tag.status.filter : 'square']">
								<div class="square-content" :style="'background-image: url('+tag.status.thumb+')'"></div>
								<div class="info-overlay-text">
									<h5 class="text-white m-auto font-weight-bold">
										<span class="pr-4">
											<span class="far fa-heart fa-lg pr-1"></span> {{tag.status.like_count}}
										</span>
										<span>
											<span class="fas fa-retweet fa-lg pr-1"></span> {{tag.status.share_count}}
										</span>
									</h5>
								</div>
							</div>
						</a>
					</div>
				</div>
				<p class="font-weight-bold text-muted mb-0">Most Recent</p>
				<div class="row">
					<div v-for="(tag, index) in tags" :key="index" class="col-4 p-0 p-sm-2 p-md-3 hashtag-post-square">
						<a class="card info-overlay card-md-border-0" :href="tag.status.url">
							<div :class="[tag.status.filter ? 'square ' + tag.status.filter : 'square']">
								<div class="square-content" :style="'background-image: url('+tag.status.thumb+')'"></div>
								<div class="info-overlay-text">
									<h5 class="text-white m-auto font-weight-bold">
										<span class="pr-4">
											<span class="far fa-heart fa-lg pr-1"></span> {{tag.status.like_count}}
										</span>
										<span>
											<span class="fas fa-retweet fa-lg pr-1"></span> {{tag.status.share_count}}
										</span>
									</h5>
								</div>
							</div>
						</a>
					</div>
					<div v-if="tags.length && loaded" class="card card-body text-center shadow-none bg-transparent border-0">
						<infinite-loading @infinite="infiniteLoader">
							<div slot="no-results" class="font-weight-bold"></div>
							<div slot="no-more" class="font-weight-bold"></div>
						</infinite-loading>
					</div>
				</div>
			</div>
			<div v-else>
				<p class="text-center lead font-weight-bold">No public posts found.</p>
			</div>
		</div>
		<div v-else class="container text-center">
			<div class="mt-5 spinner-border" role="status">
				<span class="sr-only">Loading...</span>
			</div>
		</div>
	</div>
</template>

<style type="text/css" scoped>
	.tag-header {
		font-size: 28px;
		font-weight: 300;
	}
</style>

<script type="text/javascript">
	export default {
		props: [
		'hashtag',
		'hashtagCount'
		],
		data() {
			return {
				loaded: false,
				page: 1,
				authenticated: false,
				following: false,
				tags: [],
				top: [],
			}
		},
		beforeMount() {
			this.authenticated = $('body').hasClass('loggedIn');
			this.getResults();
		},
		methods: {
			getResults() {
				axios.get('/api/v2/discover/tag', {
					params: {
						hashtag: this.hashtag,
						page: this.page
					}
				}).then(res => {
					let data = res.data;
					let tags = data.tags.filter(n => {
						if(!n || n.length == 0) {
							return false;
						}
						return true;
					});
					this.tags = tags;
					//this.top = tags.slice(6, 9);
					this.loaded = true;
					this.following = data.follows;
					this.page++;
				});
			},

			infiniteLoader($state) {
				if(this.page > (this.authenticated ? 19 : 3)) {
					$state.complete();
					return;
				}
				axios.get('/api/v2/discover/tag', {
					params: {
						hashtag: this.hashtag,
						page: this.page,
					}
				}).then(res => {
					let data = res.data;
					if(data.tags.length) {
						let tags = data.tags.filter(n => {
							if(!n || n.length == 0) {
								return false;
							}
							return true;
						});
						this.tags.push(...tags);
						if(tags.length > 9) {
							$state.complete();
							return;
						}
						this.page++;
						$state.loaded();
					} else {
						$state.complete();
					}
				});
			},

			followHashtag() {
				axios.post('/api/local/discover/tag/subscribe', {
					name: this.hashtag
				}).then(res => {
					this.following = true;
				});
			},

			unfollowHashtag() {
				axios.post('/api/local/discover/tag/subscribe', {
					name: this.hashtag
				}).then(res => {
					this.following = false;
				});
			},	

		}
	}
</script>
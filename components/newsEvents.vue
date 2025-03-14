<template>
    <section class="py-64 md:py-128 bg-pastel-orange" data-header-text="text-gray-dark">
        <div class="md:flex justify-between container inner">
            <div class="relative flex flex-col h-auto w-full md:w-1/2 text-gray-dark md:mb-0 mb-64 pb-12 border-b-2 border-gray-medium md:border-none">
                <h3>News &amp; Events</h3>
                <a v-if="$route.params.slug != 'community'" class="absolute bottom-10 hidden md:flex items-center inline--button-grey h md:text-16 text-12" href="/community">
                    View more news
                    <span class="ml-12">
                        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M23.25 12.4971H0.75" stroke="#444444" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                            <path d="M19.5 16.2471L23.25 12.4971L19.5 8.74707" stroke="#444444" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                    </span>
                </a>
            </div>
            <div v-if="total > 0" class="w-full md:w-1/2">
                <article v-for="(article, index) in paginatedArticles" :key="index" class="text-gray-dark border-b-2 border-gray-medium pb-6 mb-32">
                    <p class="text-24 leading-32 text-gray-dark h">{{article.title}}</p>
                    <div class="flex justify-start items-center">
                        <div class="tag bg-white text-gray-dark rounded-lg md:px-8 px-10 py-12 md:py-12 text-12 h my-12 capitalize mr-14 leading-none">{{article.entryTag}}</div>
                        <p class="mr-14 md:text-16 text-12">{{article.date}}</p>
                        <span>&bull;</span>
                        <a class="ml-12 flex items-center inline--button-grey md:text-16 text-12" :href="article.link">
                            See more
                            <span class="ml-6">
                                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path d="M18 13V19C18 19.5304 17.7893 20.0391 17.4142 20.4142C17.0391 20.7893 16.5304 21 16 21H5C4.46957 21 3.96086 20.7893 3.58579 20.4142C3.21071 20.0391 3 19.5304 3 19V8C3 7.46957 3.21071 6.96086 3.58579 6.58579C3.96086 6.21071 4.46957 6 5 6H11" stroke="#444444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M15 3H21V9" stroke="#444444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M10 14L21 3" stroke="#444444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                                </svg>
                            </span>
                        </a>
                    </div>
                </article>
                <div v-if="$route.params.slug == 'community'">
                  <ul>
                    <li v-for="i in totalPages" :key="i" :class="{'current': i === currentPage}" @click="currentPage = i">{{i}}</li>
                  </ul>
                </div>
                <a v-if="$route.params.slug != 'community'" class="flex md:hidden items-center inline--button-grey h md:text-16 text-12" href="/community">
                    View more news
                    <span class="ml-12">
                        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M23.25 12.4971H0.75" stroke="#444444" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                            <path d="M19.5 16.2471L23.25 12.4971L19.5 8.74707" stroke="#444444" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                    </span>
                </a>
            </div>
        </div>
    </section>
</template>
<script lang="js">
// import LinkButton from "@/assets/icons/link.svg";
export default {

    components: {
        // LinkButton
    },

    data () {
      return {
        articlesArray: [],
        total: 0,
        perPage: 5,
        currentPage: 1,
      }
    },

    computed: {
        totalPages() {
          return Math.ceil(this.total / this.perPage);
        },
        paginatedArticles() {
          const start = (this.currentPage-1) * this.perPage
          let end = this.currentPage * this.perPage
          if (end > this.total) {
            end = this.total
          }
          return this.articlesArray.slice(start, end)
        },
    },

    created() {
        this.getNewsAndEvents();
    },

    methods: {
        async getNewsAndEvents() {
            const globalData = await this.$content('events').fetch();

            let articlesFromBlog = [];
            try {
                const results = await this.$axios.$get("https://api.rss2json.com/v1/api.json?rss_url=https://blog.sigstore.dev/feed");
                articlesFromBlog = results.items;
            } catch {
                console.error("Cannot retrieve blog RSS feed");
            }

            const options = { year: 'numeric', month: 'short', day: 'numeric' };
            const articles = globalData.concat(articlesFromBlog)
                .sort((a, b) => {
                    return new Date(b["pubDate"]) - new Date(a["pubDate"])
                }).map((e) => {
                    if ("entryTag" in e === false) {
                        e["entryTag"] = "Article";
                    }
                    e["date"] = new Date(e["pubDate"]).toLocaleDateString('en', options);
                    return e;
                });

            this.articlesArray = articles;
            this.total = this.articlesArray.length
        },
    },

};
</script>
<style scoped>
ul {
  display: flex;
  align-items: center;
      justify-content: center;
      height: 100%;
}
ul li {
  padding: 1em;
  margin: 1em;
  border-bottom-width: 2px;
  border-color: transparent;
}
.current {
  font-weight: bold;
  border-color: rgba(175, 175, 175, 1);
}
</style>

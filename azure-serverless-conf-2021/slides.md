---
theme: default
colorSchema: light
background: /background.png
class: text-center
highlighter: shiki
lineNumbers: true
info: |
  ## Azure Serverless Conf
---

<div class="absolute left-9 pt-12 w-140 text-left">
  <span @click="$slidev.nav.next" class="text-3xl font-semibold rounded cursor-pointer text-gray-800">
    Serverless web applications with GraphQL, Cosmos DB and Azure Functions <ph:lightning class="inline text-yellow-500"/>
  </span>
</div>

<!--
Hello and welcome to my talk about serverless web applications with graphql, cosmosdb and azure functions. We are going to take a very brief look into the possibilities we have in Azure for a modern, microservice based infrastructure based on only serverless services
Disclaimer I hope your are not afraid of code, we will see a lot of source code in this
As we are going to look at an architecture that is based on a real world application architecture, it is not going to be
-->

---

# Speaker

<div class="flex space-x-20">
  <div>
    <img src="/jan-henrik_damaschke.jpg" class="h-40 w-40 rounded-full filter grayscale" />
    <img src="/MVP_logo_horizontal.png" class="mt-9 w-40" />
  </div>
  <div class="mt-2 flex flex-col space-y-6">
    <div class="font-bold text-2xl">Jan-Henrik Damaschke</div>
    <div>CTO / Senior Cloud Architect @ Visorian</div>
    <div>Microsoft MVP for Azure</div>
    <div><ion-logo-twitter class="inline mr-2" />@JanDamaschke</div>
    <div><ion-logo-linkedin class="inline mr-2" />/in/jan-henrik-damaschke</div>
    <div><ion-ios-paper class="inline mr-2" />itinsights.org</div>
  </div>
</div>

---

# Why serverless?

<br>
<v-clicks>

- <ph-rows class="inline" /> **Scalability** - Scale-as-you-go by time, user count, requests, etc.
- <ph-lightning class="inline" /> **Performance** - From edge workers over caching to server side rendering, everything is possible
- <ph-currency-circle-dollar class="inline" /> **Pay-as-you-go** - Pricing scales with demand
- <ph-wrench class="inline" /> **Maintenance** - Just run your code and let Azure take care of the rest
- <ph-arrow-clockwise class="inline" /> **CI/CD** - Deploy your code easier and react to changes rapidly
- <ph-arrow-clockwise class="inline" /> **Developer experience** - Separation and decoupling of services, choice of language
- <ph-arrow-clockwise class="inline" /> **Security** - Serverless has the potential to reduce your attack surface and introduce modern security standards

</v-clicks>
<br>
<br>

<!--
- Scalability: You have the choice if you want to scale, scale-out has nearly no limitations
- Performance: We have been at classic pure server side rendering to static side generators and recently prerendering with hydration. Now everything comes together in dynamic or hybrid web applications that have all the benefits of server side rendering like SEO, but also all the benefits of static generated sites, enabled through modern web standards and serverless computing. Example Nuxt Nitro rendering in browser web workers
- Pricing: Cosmos DB serverless tier, consumption tier, SaaS services and APIs
- Maintenance: The less infrastructure the better and FaaS is somewhere between SaaS and PaaS. May require new operation models, but enable rapid development and troubleshooting -> DevOps
- CI/CD: Static Web Apps is a great example for highly integrated development. GitHub PR integration, etc.
- Developer experience: Makes targeted deployments and debugging easier. Higher confidence with methodologies like chaos engineering
- Security: Smaller attack surface. Shift to other focus, more time for application related security testing like Dynamic or Static application security testing. Embraces DevSecOps and enforces good handling of secrets and config.
Azure service examples: Key Vault, App Config service, Managed Identity
-->

---

# Static Web Apps

<v-clicks>

- Seamless security model with a reverse-proxy with customizable routes
- Integrated CDN
- First-party GitHub integration
- Free SSL certificates and custom domains(yes, on domain apex)
- Built-in Authentication provider like AAD, GitHub, Twitter, etc.
- Pull requests previews
- Free basic plan for static content including Azure Functions
- Bring your own functions and custom OpenID Connect provider in standard plan
- 100 GB bandwidth limitation per subscription per month

</v-clicks>

---

# Sessionfeed.app

<v-clicks>
<Architecture class="absolute -top-5 -left-110 transform scale-50"/>
</v-clicks>

---

# Architecture overview

<v-clicks>

| Component         | Azure Service                           |
| ----------------- | --------------------------------------- |
| Frontend          | Azure Static Web Apps Vue.js            |
| Backend           | Azure Static Web Apps (Azure Functions) |
| Database          | CosmosDB (Mongo API)                    |
| Microservices     | Azure Functions                         |
| Publish/Subscribe | Azure Web PubSub                        |

</v-clicks>

---

<div class="flex">
<div>
<v-clicks>

# Why GraphQL

- Query language for APIs
- Shared definitions of resources
- Strongly connected entities coming from multiple APIs
  Central API Gateway
- Works great in serverless
- Authorization and Authentication support
- Rich ecosystem

</v-clicks>
</div>
<div>
<v-clicks>

# Limitations with serverless

- No subscription support
- GraphQL API only usable with bring your own app in Static Web Apps
- Performance (kind of)
- Authentication/Authorization can be difficult

</v-clicks>
</div>
</div>

---

# Cosmos DB

<div class="text-4xl text-center mt-20" v-if="$slidev.nav.currentPage === 8" v-motion-slide-top>
<wpf-full-trash class="inline align-bottom" /><span class="mx-8">Demo Time</span><wpf-full-trash class="inline align-bottom" />
</div>

---

# Session API

sessionApi/getComment (simplified)

```ts {1|2-10|11,12}
const { id } = req.query;
const question: Question = await makeMongoQuery(
  sessionfeedbDbName,
  questionsCollectionName,
  "findOne",
  [{ _id: id }],
  0,
  mongoConfiguration,
  cacheConfiguration
);
context.res = {
  body: question : undefined,
};
```

---

# GraphQL Datasource

graphql/datasources/SessionAPI (simplified)

```ts {}
export class SessionAPI extends RESTDataSource {
this.baseURL = sessionApiBaseUrl;

willSendRequest(request: RequestOptions) {
  request.headers.set("x-functions-key", sessionApiFunctionKey);
}

async getQuestion(id: string): Promise<Question> {
  const result = await this.get("/getQuestion", {
    id,
  });
  return this.convertQuestionFields(JSON.parse(result));
}
...
```

---

# GraphQL typing

<div class="flex justify-around space-x-2">

```graphql {0|3,10|4,6,7|all}
type Question {
  id: ID!
  comments: [Comment]!
  commentCount: Float!
  created: DateTime!
  likeCount: Float!
  liked: Boolean!
  likedBy: [String!]!
  modified: DateTime!
  talk: Talk!
  ...
}
```

```ts {0|1-8|9-13|all}
export class Question {
  @Field((type) => ID)
  id: string;
  @Field({ nullable: "items" })
  comments: Comment[];
  @Field()
  created: Date;
  @Authorized(["ADMIN"])
  @Field()
  talkId: string;
  @Authorized(["ADMIN"])
  @Field()
  modified: Date;
}
```

```ts {0|all}
@Resolver((of) => Question)
export class QuestionResolver {
  ...
  @FieldResolver()
  likeCount(@Root() question: Question): number {
    return question.likedBy ?
      question.likedBy.length : 0;
  }
  ...
}
```

</div>

---

# Authentication / Authorization

<div class="flex justify-around">

```json {2|4-7|11-13|17,18}
{
  "routes": [
    {
      "route": "/talks/*",
      "allowedRoles": [
        "authenticated",
        "admin",
      ]
    },
    {
      "route": "/api/*",
      "allowedRoles": [
        "authenticated"
      ]
    },
    {
      "route": "/login/github",
      "rewrite": "/.auth/login/github"
...
```

```json {2-4|7-14|15-19}
    {
      "route": "/logout",
      "redirect": "/.auth/logout",
      "statusCode": 301
    }
  ],
  "navigationFallback": {
    "rewrite": "index.html",
    "exclude": [
      "/@vite/*",
      "/assets/*",
      "/.vite/*"
    ]
  },
  "responseOverrides": {
    "401": {
      "redirect": "/login",
      "statusCode": 302
    }
...
```

</div>

---

# Access user information

<div class="flex justify-around space-x-2">
  <div>
  <span>Frontend</span>

```ts {0|all|0}
const {
  data: swaUser,
  isFetching: swaUserLoading,
  error: swaUserError,
  execute: refetchSwaUser,
  onFetchResponse,
} = useFetch("/.auth/me").get();
```

```ts {0|all|0}
async function getUserInfo() {
  const response = await fetch("/.auth/me");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}
```

  </div>
  <div>
  <span>Backend</span>

```ts {0|all|0}
const context = async ({ request }: any) => {
  try {
    const header = request.headers["x-ms-client-principal"];
    const encoded = Buffer.from(header, "base64");
    const decoded = encoded.toString("ascii");
    const swaUser: SwaUser = JSON.parse(decoded);
    return { swaUser: swaUser };
  } catch (error) {
    throw new AuthenticationError(
      "This API can only be accessed through Azure Static Web Apps"
    );
  }
};
```

  </div>
</div>

---

# What else?

- Durable Function -> Next talk
- No config management
- No secrets management
- No premium plans
- No caching
- Advanced architecture

<!--
- Durable functions: polling consumer pattern
- Azure App Configuration DevSecOps
- Azure Functions cold starts, CosmosDB serverless scaling
- Redis, Azure Storage, etc.
- Azure Function in AKS, Dapr, Serverless workers Container instances with KEDA scale in to 0 nodes
-->

---

# Takeaways

- Know your workloads -> Scalability targets
- Know your culture -> Dev(Sec)Ops
- Know your code -> Testing, testing testing
- Know your risks -> Think security from the beginning on
- Know your cloud platform -> Mix FaaS and PaaS services
- Know your environment -> Think about local development and testing
- Know your budget -> Invest time to plan for scalable service and configure alerts (e.g. CosmosDB)

<!--
Mix FaaS, PaaS, SaaS to meet scalability and reliability targets
Cosmos DB 0,26€ Pro 1 Mio. RUs
Function 130€ 1 core
Function 260€ 2 cores
Function 520€ 4 cores
Redis 13,54€ 250MB
-->

---

# Questions?

## Ask your questions at [https://sessionfeed.app](https://sessionfeed.app)

<br>

## If you want to create great development presentations like this one, use [https://sli.dev](https://sli.dev) by Anthony Fu @antfu7

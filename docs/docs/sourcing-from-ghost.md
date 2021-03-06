---
title: Sourcing from Ghost
---

[Ghost](https://ghost.org) is an open source, professional publishing platform built on a modern Node.js technology stack — designed for teams who need power, flexibility and performance - used by Apple, NASA, Sky News, OpenAI & many more.

It comes with all the benefits of modern, centralised Headless CMS platforms, with the added benefit of being released completely for free under an MIT license, so you have total ownership and control of it without needing to depend on a third party back-end.

This guide will walk you through using [Gatsby](/) with the [Ghost Content API](https://docs.ghost.org/api/content/).

&nbsp;

---

## Quick start

The fastest way to get started is with the official **Gatsby Starter Ghost** repository, which contains a light scaffolding of queries and templates to get a brand new site up and running.

- Repository: https://github.com/tryghost/gatsby-starter-ghost
- Demo: https://gatsby.ghost.org

[![Gatsby Starter Ghost](./images/gatsby-starter-ghost.jpg)](https://gatsby.ghost.org)

&nbsp;

---

## Install & setup

If you prefer to start from scratch or integrate the Ghost Content API into an existing site, you can set up the **Gatsby Source Ghost** plugin.

```shell
npm install --save gatsby-source-ghost
```

### Configuration

```javascript:title=gatsby-config.js
// These are working demo credentials, try them out!

module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-ghost`,
      options: {
        apiUrl: `https://gatsby.ghost.io`,
        contentApiKey: `9cc5c67c358edfdd81455149d0`,
      },
    },
  ],
}
```

&nbsp;

---

## Generating pages

Once the source plugin is set up, you can use the `createPages` API in `gatsby-node.js` to create queries on your Ghost data with GraphQL. In this example, Gatsby iterates over each post returned by the Ghost API and generates a new page with that data, using the `post.js` template file.

There are several ways to structure queries depending on how you prefer to work, but here's a very minimal example:

```javascript:title=gatsby-node.js
exports.createPages = ({ graphql, actions }) => {
  const { createPage } = actions
  const createPosts = new Promise((resolve, reject) => {
    const postTemplate = path.resolve(`./src/templates/post.js`)

    resolve(
      graphql(`
        {
          allGhostPost(sort: { order: ASC, fields: published_at }) {
            edges {
              node {
                slug
              }
            }
          }
        }
      `).then(result => {
        const items = result.data.allGhostPost.edges

        _.forEach(items, ({ node }) => {
          node.url = `/${node.slug}/`
          createPage({
            path: node.url,
            component: path.resolve(postTemplate),
            context: {
              slug: node.slug,
            },
          })
        })

        return resolve()
      })
    )
  })

  return Promise.all(createPosts)
}
```

&nbsp;

---

## Outputting data

Then, within the `post.js` template, you can determine exactly how and where you want to output data on each page. Again, you'll use GraphQL to query individual fields, so a simple example looks something like this:

```javascript:title=templates/post.js
import React from "react"
import { graphql } from "gatsby"

const Post = ({ data }) => {
  const post = data.ghostPost
  return (
    <>
      <article className="post">
        {post.feature_image ? (
          <img src={post.feature_image} alt={post.title} />
        ) : null}
        <h1>{post.title}</h1>
        <section dangerouslySetInnerHTML={{ __html: post.html }} />
      </article>
    </>
  )
}

export default Post

export const postQuery = graphql`
  query($slug: String!) {
    ghostPost(slug: { eq: $slug }) {
      id
      title
      slug
      feature_image
    }
  }
`
```

&nbsp;

---

## Wrapping up

You should have a broad understanding of how Gatsby and the Ghost Content API work together now in order to use Ghost as a headless CMS. Your writers can enjoy the slick administration experience, while your development team can keep using their ideal tooling. Everyone wins!

### What to read next

Here are some further resources and reading material to help you get started with some more advanced examples and use-cases:

- [Gatsby + Ghost announcement post](/blog/2019-01-14-modern-publications-with-gatsby-ghost/)
- [More info about Ghost as a Headless CMS](https://blog.ghost.org/jamstack/)
- [Official Gatsby Starter for Ghost](https://github.com/tryghost/gatsby-starter-ghost)
- [Official Gatsby Source Plugin for Ghost](/packages/gatsby-source-ghost/)
- [Official Ghost developer docs](https://docs.ghost.org/api/)

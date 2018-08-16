# Creating a contentful and Gatsby site

## Final Objectives

* Create a page for every content on the contentful platform
* Render the data inside the components

## Step 1: Create a testing space on Contentful

For this example we will start with a contenfull pre made space, that already contains Content models and Contents so we can test our app

## Step 2: Create a basic Blog

I will be using the starter project [https://github.com/contentful-userland/gatsby-contentful-starter](Source)
In the setup of the project be aware of the order of the API keys.
Once the project is ready you should see a new simple Blog.

## Step 3: Creating a page for every content: "Course"

We will add a new query to Courses inside the exisitng one for blog posts

We will be using the Async plugins from the Node API, because we need to create pages getting the data from an API request, the fuction must return a Promisse in order to execute the queries correctly.

We will use the [createPages](https://www.gatsbyjs.org/docs/node-apis/#createPages)

example:

```js
const path = require("path");

exports.createPages = ({ graphql, boundActionCreators }) => {
  const { createPage } = boundActionCreators;
  return new Promise((resolve, reject) => {
    const blogPostTemplate = path.resolve(`src/templates/blog-post.js`);
    // Query for markdown nodes to use in creating pages.
    resolve(
      graphql(
        `
          {
            allMarkdownRemark(limit: 1000) {
              edges {
                node {
                  fields {
                    slug
                  }
                }
              }
            }
          }
        `
      ).then(result => {
        if (result.errors) {
          reject(result.errors);
        }

        // Create blog post pages.
        result.data.allMarkdownRemark.edges.forEach(edge => {
          createPage({
            path: `${edge.node.fields.slug}`, // required
            component: slash(blogPostTemplate),
            context: {
              slug: edge.node.fields.slug
            }
          });
        });

        return;
      })
    );
  });
};
```

That function will get the data from the API and for each will create a new page, we need to specify:

1.  Template, in wich component we want the data to go to be rendered, inside the template component will be only one course data.

```js
const blogPost = path.resolve("./src/templates/blog-post.js");
const courseTemplate = path.resolve("./src/templates/course.js");
```

2.  Modifying the grapql query: Using GraphQl we can get all the nodes in the DB in one call, using gatsby-source-contentful the main query is made using `allContentful + {Mode}` and we add them after other:

```js
{
  allContentfulCourse {
    edges {
      node {
        title
        slug
      }
    }
  }
  allContentfulBlogPost {
    edges {
      node {
        title
        slug
      }
    }
  }
}
```

3.  Creating the pages: If the function resolves correctlty we will parse the createPage function with 3 arguments

```js
const courses = result.data.allContentfulCourse.edges;
courses.forEach((course, index) => {
  createPage({
    path: `/course/${course.node.slug}/`,
    component: courseTemplate,
    context: {
      slug: course.node.slug
    }
  });
});
```

4.  Checking if the pages were created: reestart the server and go to any undefined path, on the dev mode gatsby will show all the paths available, that should include all the courses page

```js
const Promise = require("bluebird");
const path = require("path");

exports.createPages = ({ graphql, boundActionCreators }) => {
  const { createPage } = boundActionCreators;

  return new Promise((resolve, reject) => {
    const blogPost = path.resolve("./src/templates/blog-post.js");
    const courseTemplate = path.resolve("./src/templates/course.js");
    resolve(
      graphql(
        `
          {
            allContentfulCourse {
              edges {
                node {
                  title
                  slug
                }
              }
            }
            allContentfulBlogPost {
              edges {
                node {
                  title
                  slug
                }
              }
            }
          }
        `
      ).then(result => {
        if (result.errors) {
          console.log(result.errors);
          reject(result.errors);
        }

        const posts = result.data.allContentfulBlogPost.edges;
        posts.forEach((post, index) => {
          createPage({
            path: `/blog/${post.node.slug}/`,
            component: blogPost,
            context: {
              slug: post.node.slug
            }
          });
        });

        const courses = result.data.allContentfulCourse.edges;
        courses.forEach((course, index) => {
          createPage({
            path: `/course/${course.node.slug}/`,
            component: courseTemplate,
            context: {
              slug: course.node.slug
            }
          });
        });
      })
    );
  });
};
```

5.  Create the template course.js inside the templates folder

6.  Creating a query to get all the data on the page

To get the page content we will query the DB using graphQl and filter the the call using the slug, so after the export function on the component we will add a query to get only the title for now:

```js
export const pageQuery = graphql`
  query CoursesBySlug($slug: String!) {
    contentfulCourse(slug: { eq: $slug }) {
      title
    }
  }
`;
```

Now inside the CourseTemplate component we will have another prop call data:

```js
data:{
    contentfulCourse:{
        title:  "Hello Contentful"
    }
```

We can set that inside the component using lodash/get:

```js
import get from "lodash/get";
```

```js
class CourseTemplate extends React.Component {
  render() {
    const post = get(this.props, "data.contentfulCourse");
    const siteTitle = get(this.props, "data.site.siteMetadata.title");
    return (
      <div>
        <Helmet title={`${post.title} | ${siteTitle}`} />
        <h1 className="section-headline">{post.title}</h1>
      </div>
    );
  }
}
```

7.  Set HTML content: To set a html content we can't use the default props because react will understand that as a plain text, we need to use a property of JSX called `dangerouslySetInnerHTML`, and to get the content as a HTML we will add a plugin `gatsby-transformer-remark` on the gatsby-config.js, then inside the grapQl query we will have the property `childMarkdownRemark` that should gave us the html for the component

```js
export const pageQuery = graphql`
  query CoursesBySlug($slug: String!) {
    contentfulCourse(slug: { eq: $slug }) {
      title
      childContentfulCourseDescriptionTextNode {
        childMarkdownRemark {
          html
        }
      }
    }
  }
`;
```

In the component:

```js
<div
  dangerouslySetInnerHTML={{
    __html:
      post.childContentfulCourseDescriptionTextNode.childMarkdownRemark.html
  }}
/>
```

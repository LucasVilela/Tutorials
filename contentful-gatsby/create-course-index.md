# Create a List of courses

1.  Create a new page `courses` on the pages folder

2.  Adding a pageQuery that receives all the post

* Excerpt stands for a piece of the data accept ```(pruneLength: 300)```
[Reference](https://using-remark.gatsbyjs.org/excerpts/)

```js
export const pageQuery = graphql`
  query CourseIndexQuery {
    allContentfulCourse {
      edges {
        node {
          title
          slug
          description {
            childMarkdownRemark {
              excerpt
            }
          }
        }
      }
    }
  }
`;
```

#### Filtering query:

Locale:

```js
allContentfulCourse(filter: { node_locale: { eq: "en-US" } });
```

Sort by created:

```js
allContentfulBlogPost((sort: { fields: [publishDate], order: DESC }));
```

3.  Parsing the data inside the component

```js
class CourseIndex extends React.Component {
  render() {
    const courses = get(this.props , "data.allContentfulCourse.edges")
    return <div>
        <h1>Courses Index</h1>
        {courses.map(({node})=>{
            return(
            <li key={node.slug}>
                <Link to={`${node.slug}`}> {node.title} </Link>
                <div dangerouslySetInnerHTML={{__html: node.description.childMarkdownRemark.excerpt}}/>
            </li>)
        })}
    </div>
  }
}

```

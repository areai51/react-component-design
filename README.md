# Components Design Guide for React [Very very WIP]
There are multiples approaches to building components in react, and each of them have their own pros and cons.
The aim of this document is to set some very high guiding principles for developing React Components, that would allow for things like re-usability, scalability and performance. 


## Table of Contents

* [Applying Atomic Design Principles to Component](#applying-atomic-design-principles-to-component-design) 
* [How Re-Usable should your Components be?](#how-reusable-should-your-components-be)
* [The Skeleton and Skins Approach to Styling Components](#the-skeleton-and-skins-approach-to-styling-components)
* [Leverage Higher Order Components (HOC)](#leverage-higher-order-components)


### Applying Atomic Design Principles to Component Design 
While the components and container approach sets a broad guideline to building components, we feel following Atomic Design Principles can help teams be more consistent in their approach to building components.
>Go Pure and Stateless as much as you can.

### Atoms
Atoms are regular DOM elements with the necessary CSS class names and ARIA properties defined. 

While there are multiple schools of thought that advocate creating a React component for Atoms like H1, Input, Link tags etc. we feel it is unnecessary additional overhead we can live without.

An Atom would not have an existence on its own and will always be used within a React Molecule. The primary role of an Atom is to ensure consistency in style and accessibility compliance throughout the application.
Atoms will be responsive in nature.

### Molecules
Molecules will form the smallest unit of React Components. A molecule may consist of two or more Atoms or an Atom and molecule within it.
Molecules are stateless React Components and will display data via props. They should be pure functional components. There are a [numerous benefits in keeping these as pure functional components.](https://hackernoon.com/react-stateless-functional-components-nine-wins-you-might-have-overlooked-997b0d933dbc#.t54zao6qp)
Molecules will be responsive in nature and will change shape (read height and width) based on the container it is wrapped in. Molecules should contain styling that only defines its structure or skeleton.
Avoid building components that do too many things.
Here is an example for a Button molecule.

```javascript
const Button = (props) => {
  return (
          <button
      type="button"
      onClick={props.onClick}
      className={styles.primary}
    >
      {props.children}
    </button>
  );
}
Button.propTypes = {
  children: PropTypes.string
};
export default Button;
```

Another example of the molecule is the Card component

```javascript
const Card = (props) => {
  const cardDetails = (props.cardData !== null) ? props.cardData : null;
  return (
    <div className={styles.cardContainer}>
      {cardDetails && cardDetails.map((card, index) =>
        <div className={styles.card} key={`key-${index}`}>
          <img src={card.image} alt={card.heading}/>
          <h2 children={card.heading} />
          <Rating value={4} />
          <p>{card.text} </p>
          <Button> Add to Cart</Button>

        </div>
      )}
    </div>
  );
};

Card.propTypes = {

  cardData: PropTypes.arrayOf(PropTypes.shape({}))
};

export default Card;

```

### Organisms
Organisms are stateful React Components. An Organism is a collection of stateless React Molecules. Organisms  contain state and will pass it down to the molecules within it.
Organisms could be adaptive in nature. i.e an organism would change shape and form based on the device or screen-size it is being viewed in. eg: Navigation bar. Organisms would contain styles that only apply the wireframe or skeleton of the component. It should not apply any visual styling in terms of color and look and feel to the organism.

The CardContainer organism would look something as follows:

```javascript
class CardContainer extends Component {
  componentWillMount() {
    this.props.fetchCatelogCards();
  }

  render() {
    const { cardData } = this.props;
    const data = (Object.keys(cardData).length) === 0 ? null : cardData.data;

    return (
      <div>{<Card cardData={data}/>}</div>
    );
  }
}

CardContainer.propTypes = {
  cardData: PropTypes.object,
  fetchCatelogCards: PropTypes.func
};
export default CardContainer;

```

### Templates
Templates are a collection of stateful React Organisms. They would also be Higher Order Components that use the concept of Composition to build components.

Templates would be responsive and device aware and will pass information to load the relevant Adaptive Organisms based on the device viewport.
Templates have a route associated with it, and would allow Webpack to do code splitting based on the routes.
Ideally each template should be bundled as an individual JS bundle.

```javascript
const mapStateToProps = (state) => {
  return {
    cardData: state.CardContinerPage.cardData
  };
};

const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    fetchCatelogCards
  }, dispatch);
};

export default connect(mapStateToProps, mapDispatchToProps)(CardContainer);
```

### Pages
Pages are templates with route parameters. Pages would also contain styles that will apply the ‘skin’ to the entire page. A skin essentially comprises of look and feel, aesthetics of the page. 


## How Reusable should your Components be
In our quest to make our components re-usable, we start to build in functionality that tries to cover all possible use cases that the component can be used in. While doing so we land up over engineering our components. We have examples of button components that also act as hyper link, or an Accordion component that can render either vertically or horizontally.

Here is an example from react bootstrap , where a button component work works as a hyperlink. This is an example of something one must avoid.

```javascript
// BAD 
render() {
    const { active, block, className, ...props } = this.props;
    const [bsProps, elementProps] = splitBsProps(props);

    const classes = {
      ...getClassSet(bsProps),
      active,
      [prefix(bsProps, 'block')]: block,
    };
    const fullClassName = classNames(className, classes);

    if (elementProps.href) {
      return this.renderAnchor(elementProps, fullClassName);
    }

    return this.renderButton(elementProps, fullClassName);
  }
}
```

>A component should not be doing multiple things.

Follow these basic rules while building your component.

1. If the render method of your component contains 10 lines or more, it is a signal to break it down.
1. If the markup for one use case is nearly equal to the markup that renders the other use case, then it is best to break them out as two separate stateless components and import them into a wrapper component.
1. Your re-usable component should contain code that covers the most common use cases. There-by when another developer extends your component to add some additional functionality, they are not inheriting unnecessary bloat.  

> Remember the people using your component are developers and are skillful enough to modify/extend your component they don't need to be given everything on a silver plate.


## The Skeleton and Skins Approach to Styling Components

There are multiple approaches to styling components in React. We have PostCS, CSS Modules, Styled Components etc. 

Most of these solutions advocate a single class approach, meaning all the styles for the component are added via a single class. While this is good for most cases and the fact that React components built following the Atomic Design Pattern offer a more powerful construct of styling, we feel it is important to separate the styles that provide structure from the ones that provide look and feel.

Think of the structural styles as skeleton that are associated with each component. The skin consists of styles that is applied at a page or template level that will apply the consistent look and feel for all the components on that page.

De-coupling the skin styles from the skeleton styles, allows re-usability of components and the ability to theme them with minimal CSS overrides.

A couple of reasons you may want to do this are:
* You may want to run an A/B test with different colored buttons or headings.
* You'd like to theme a page for say St. Patrick's day or Halloween.
* You may have brand sponsored pages that need to follow the brand's theme guidelines.
* Or You might just want to re-use that component in a totally different project.


## Leverage Higher Order Components 
By Definition, a Higher Order Component (HOC) is a function that takes in a component and returns a new component. This is a great way to add features to an existing component without necessarily having to extend it. 
HOCs allow you to do the following

* Abstract and Manipulate State.
* Manipulate Props.


HOC is also a great way to abstract data sources and state from the component.

```javascript
const container = endpoint => Card => 
 class extends Component {
 
 state = {
   data: []
 }
 
 componentDidMount() {
   fetch(endpoint)
     .then(response => response.json())
     .then(data => this.setState({ data }))
 }
 
 render() {
   return <Card data={this.state.data} />
 }
}

```

The higher order component called CardContainer would look something like this.
```javascript
const CardContainer = container(
  ‘data/catalog'
)(Card)
```

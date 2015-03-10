# Comment

```javascript
var converter = new Showdown.converter();
var Comment = React.createClass({
  render: function() {
    var rawMarkup = converter.makeHtml(this.props.children.toString());
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});
```
### Q1: Where does the value of ``this.props.author`` get specified?

In HTML attributes. So if you were to use this class, you'd use ``<Comment author="some value" />``
And that's where ``this.props.author`` gets set.

### Q2: Where does the value of ``this.props.children`` get specified?

It's the children of the Comment class. So if we were to use ``<Comment > stuff </Comment>``
stuff would be ``this.props.children``, everything that gets wraped by Comment element, is children.


### Q3: What does ``className="comment"`` do?

It is basically assigning CSS classes but in JSX.

### Q4: What is ``dangerouslySetInnerHTML``? Why is it such a long word for an API method?

It is long because it is not meant to be used unless in special occasions where doing otherwise is hard. Note that using this API is insecure, it may lead to an XSS attack. It basically let's you put dynamic raw html markups. Which means that someone could inject HTML elements referencing a javascript code that could potentially be harmful to users. In certain situations, however, using this makes sense.

# CommentBox
```javascript
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this),
      error: function(xhr, status, err) {
        console.error(this.props.url, status, err.toString());
      }.bind(this)
    });
  },

```

### Q5: How does ``$`` get defined? Is it part of the ReactJS framework?

No it's provided by jQuery. it's a basic ajax call.

### Q6: Where does the value of ``this.props.url`` get specified?

Same as question one, it gets set from the parent using HTML params or XML attributes.


### Q7: What would happen to the statement ``this.setState`` if ``bind(this)`` is removed? Why?

Binding is for giving context, if you don't bind the function with your current object instance (aka this) then you won't be able to use this.props and/or this.state or such contextual values inside that function. 

### Q8: Who calls ``loadCommentsFromServer``? When? 

it gets called from 'componentDidMount' once and then it get's called every 'pollInterval' seconds. because it's scheduled to get called.


```javascript
	
  handleCommentSubmit: function(comment) {
    var comments = this.state.data;
    comments.push(comment);
    this.setState({data: comments}, function() {
      // `setState` accepts a callback. To avoid (improbable) race condition,
      // `we'll send the ajax request right after we optimistically set the new
      // `state.
      $.ajax({
        url: this.props.url,
        dataType: 'json',
        type: 'POST',
        data: comment,
        success: function(data) {
          this.setState({data: data});
        }.bind(this),
        error: function(xhr, status, err) {
          console.error(this.props.url, status, err.toString());
        }.bind(this)
      });
    });
  },
  getInitialState: function() {
    return {data: []};
  },
  componentDidMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm onCommentSubmit={this.handleCommentSubmit} />
      </div>
    );
  }
});
```

### Q9: What is the purpose of ``this.state``? How is this different from ``this.props``?

State is different Properties, properties are given to the class from parent. State is owned by the class itself not its parent. It is private to the class and can only be changed by calling this.setState. This should be used when we need a mutable object. Properties, however, are owner by parents and the React class is only supposed to render according to the given property.

### Q10: What is the initial value of ``this.state.data``? How is the initial value specified?

It is initialized as empty array.
In the class, implement 'getInitialState' and return an object, this object is your initial state.


### Q11: What is the new value of ``this.state.data``? How is this new value set?

It can be changed by calling ``this.setState(<state object>);``


### Q12: What is the purpose of ``componentDidMount`` callback?

If there are things need to be done the first time the component get mount, you put those in 'componentDidMount'.
Note that, rerendering will not call this function.


### Q13: What is the purpose of ``getInitialState``?

To set the initial state of your states. In this example, you need to set a default value for your 'data' so you can work with your states.


# CommentList

```javascript

var CommentList = React.createClass({
  render: function() {
    var commentNodes = this.props.data.map(function(comment, index) {
      return (
        // `key` is a React-specific concept and is not mandatory for the
        // purpose of this tutorial. if you're curious, see more here:
        // http://facebook.github.io/react/docs/multiple-components.html#dynamic-children
        <Comment author={comment.author} key={index}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});
```
### Q14: How does the value of ``this.props.data`` get set?

It is automatically updated by React, it gets set from parent's state. when that state is changed, React will automatically change the props of the children elements.

### Q15: What is the value of ``commentNodes``?

It is an array of ReactElement's that will later get compiled to HTML.

### Q16: Where does the value of ``{comment.text}`` go on the rendered page?

Since it's the children for Comment class.
It'll be the chilren of a div. The div that is rendered version of a Comment class.

# CommentForm
```javascript

var CommentForm = React.createClass({
  handleSubmit: function(e) {
    e.preventDefault();
    var author = this.refs.author.getDOMNode().value.trim();
    var text = this.refs.text.getDOMNode().value.trim();
    if (!text || !author) {
      return;
    }
    this.props.onCommentSubmit({author: author, text: text});
    this.refs.author.getDOMNode().value = '';
    this.refs.text.getDOMNode().value = '';
  },
  render: function() {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input type="text" placeholder="Your name" ref="author" />
        <input type="text" placeholder="Say something..." ref="text" />
        <input type="submit" value="Post" />
      </form>
    );
  }
});

React.render(
  <CommentBox url="comments.json" pollInterval={2000} />,
  document.getElementById('content')
);
```

### Q17: What is the purpose of ``e.preventDefault()``?

It prevents the browser from doing the default behaviour which is refreshing the page.

### Q18: What is the value of ``this.props.onCommentSubmit``? What does it get specified?

Gets specified when it's parent is creating this class. So CommentBox sets that.

### Q19: Where does ``this.refs.author`` point to?

points to the ReactElement who has ``'ref=author'``.

### Q20: What does ``getDOMNode()`` do?

It returns the HTML Document node attached to the React class.

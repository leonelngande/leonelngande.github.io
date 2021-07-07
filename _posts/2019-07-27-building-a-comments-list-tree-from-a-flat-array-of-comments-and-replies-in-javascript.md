---
layout: post
hidden: false
title: >-
  Building a Comments List Tree from a Flat Array of Comments and Replies in
  Javascript
tags: []
---
```javascript
/**
 * @see https://stackoverflow.com/a/18018037/6924437
 * @param comments
 * @param parentIdKey
 */
listToTree(comments: IComment[], parentIdKey = 'parent_id'): IComment[] {
  // const commentsCopy = [...comments];
  const mapLookup = {};
  let node: IComment;
  const roots: IComment[] = [];

  const commentsCopy = comments.map((comment, index) => {
    const commentCopy = {...comment};
    mapLookup[commentCopy.id] = index; // initialize the map
    commentCopy.children = []; // initialize the children
    return commentCopy;
  });

  commentsCopy.forEach((comment, index) => {
    node = comment;
    if (node[parentIdKey]) {
      // if you have dangling branches check that map[node.parentId] exists
      commentsCopy[mapLookup[node[parentIdKey]]].children.push(node);
    } else {
      roots.push(node);
    }
  });
  
  return roots;
}
```

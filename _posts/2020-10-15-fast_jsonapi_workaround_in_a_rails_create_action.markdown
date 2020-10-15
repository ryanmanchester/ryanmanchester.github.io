---
layout: post
title:      "FAST_JSONAPI Workaround in a Rails #CREATE Action"
date:       2020-10-15 18:28:53 +0000
permalink:  fast_jsonapi_workaround_in_a_rails_create_action
---


I am currently working on my final portfolio project for the Flatiron School online curriculum(Yay!). I'm building a small e-commerce site geared toward fashion designers, vintage clothing re-sellers, or basically anyone wanting to sell some one-of-a-kind items. Given all the associations and data that make up an individual item and a seller account, I decided to use fast_jsonapi to serialize the data before sending it to my frontend. This allows the programmer to control what makes it to the frontend to display to the user.  Enough backstory. If you clicked here after reading that atrocious title, then you know what a serializer does. So let's get on with dealing with this error in fast_jsonapi.

First, let's make sure the serializer itself is set up properly. If you follow the Getting Started guide for fast_jsonapi, then by now you've put the gem in your Gemfile and bundled, and used the Rails generator to create the serializers file in your app directory. Here's what mine looks like after doing all that and writing out the associations and attributes I want to display:
```
class ItemSerializer
  include FastJsonapi::ObjectSerializer
  attributes :name, :size, :description, :price, :image_url, :seller_id

  belongs_to :category
  belongs_to :seller

end
```

Pretty standard. Now on to the items_controller.
```
 def index
    category = Category.find_by(id: params[:category_id])
    items = category.items
    render json: ItemSerializer.new(items)
  end

  def create
    item = current_seller.items.build(item_params)
    if item.save

      render json: ItemSerializer.new(item)
    else
      render json: {
        error: item.errors.full_messages.to_sentence
      }
    end
  end
```
Now I've included Item#index to make a point: it works as pictured. It serializes the list of items as separated by categories. Let's take Item#create line by line. We build a new item associated to the current_seller(seller that is logged in) with the whitelisted params. Standard Rails. If the item is valid and is saved to our database (and thus created), we want to send some serialized JSON back as a response to our frontend. Let's run it:

```
serialized_item = ItemSerializer.new(item)
#<ItemSerializer:0x00007fcce1b90c70 @fieldsets={}, @params={}, @resource=#<Item id: 51, name: "Evening dress", description: "Beige, long sleeve evening dress", size: "4", image_url: "https://images.pexels.com/photos/4694316/pexels-ph...", price: 110.0, sold: false, created_at: "2020-10-15 14:52:24", updated_at: "2020-10-15 14:52:24", category_id: 2, seller_id: 3>>
```

Ok, the serializer seems to be working. I know because I've checked this format against the format in Item#index. They are identical. So lets render the json: 

```
serialized_item.serialized_json
*** NoMethodError Exception: undefined method `each' for #<Item:0x00007fcce4cbb430>

nil
```

What the hell?! Why does it work flawlessly when fetch the items from the backend, but throws this weird error when creating. Because the serializer is expecting an array. It's trying to over the object using 'each.' Can't do it. So...here's my workaround. 

I wrapped the item variable I'm passing to ItemSerializer as an array:

```
  def create
    item = current_seller.items.build(item_params)
    if item.save

      render json: ItemSerializer.new([item])
    else
      render json: {
        error: item.errors.full_messages.to_sentence
      }
    end
  end
```

This allows it to serialize. But it comes with an additional top layer called 'data.' It's an array referred to as 'data' containg the new object. This is what is coming up to the frontend response. We want to structure all the data in our Redux store in the same manner, so we need to access the object to concatinate it to our items array in our Redux store. 

```
fetch(`http://localhost:3000/api/v1/categories/${itemData.category}/items/${itemData.itemId}`, {
      credentials: 'include',
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(sendItem)
    })
    .then(resp => resp.json())
    .then(item => {
        if (item.error) {
          alert(item.error)
        } else {
          const newItem =  item.data[0]
          dispatch(addNewItem(newItem))
          dispatch(clearItemForm())
          history.push(`/sellers/${newItem.attributes.seller_id}/items/${newItem.id}`)
        }


    })
```
		
Since we now have access to the object, we can just concatinate in the regular Redux way:
		
```
 case 'ADD_NEW_ITEM':
      const newItem = state.included.concat(action.item)
        return {
          ...state,
          included: newItem
        }
```

That's it! 

To recap, we had an issue of getting fast_jsonapi to serialize our data in our Item#create action. We wrapped our item in an array and passed that to our ItemSerializer. Then we were able to access the object from the array and dispatch the objec itself to the reducer then add it to Redux.  Now all of our items are of the same data structure:

```
17: {id: "60", type: "item", attributes: {…}, relationships: {…}}
18: {id: "61", type: "item", attributes: {…}, relationships: {…}}
19: {id: "62", type: "item", attributes: {…}, relationships: {…}}
20: {id: "63", type: "item", attributes: {…}, relationships: {…}}
21: {id: "63", type: "item", attributes: {…}, relationships: {…}}
```
*20 & 21 are from the create action. 17-19 were stored via another reducer*

Thanks for reading and I hope this saves you the hours I lost! 
		


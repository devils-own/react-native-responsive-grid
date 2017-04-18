
# Responsive Grid for React Native

## Why?

Developing performant, responsive and fairly detailed 2D layouts with raw flexbox in React Native (prior to v0.42) used to take me hours per screen and resulted in markup and styles that were almost unmaintainable. While Flexbox itself maybe confusing to new comers, the lack of perofmant way to encode relative size info was the real pain. Then came React Native v0.42 and solved that problem. Since then there have been several folks who have made flexbox based responsive grids. I've taken one of those, namely, `react-native-flexbox-grid` by @rundmt, which is based on yet another library, `react-flexbox-grid`, and I made some major changes and enhancements to fit what I believe is a simpler mental model for 2D responsive layout, based purely on percentages.

## What?

This Responsive Grid (for React Native) fixes the calculation model for Grid based layouts by abandoning the format-based, columns-per-view approach (e.g. "12 column grid") and instead allowing the developer to specify the width of each grid column as a percentage of parent view's size, so 10% meams 10 column grid. There is no reason for a grid to be 11, 12, 14 or 16 columns. Grids can be any number of columns, including a fractional number, and that number should be determined only by the desired width per column, which determines the standard offset for placing things, including text. You can think of it as specifying the grid resolution using percentages.

One constraint this version of the grid has to keep its structure and design simple is that Rows may not contain other Rows as children (they must be wrapped in a Column) and Columns may not contain other columns as children (they must be wrapped in a Row) 

I've also found that RTL (right-to-left) support (for Hebrew/Arabic apps) to be generally lacking in RN, so I added RTL layout support to this version. 

## Terms:

RTL = right-to-left layout (Hebrew/Arabic)
LTR = "normal" left-to-right layout

## Relative and Responsive Layout

```
import {Column as Col, Row} from 'react-native-responsive-grid';

<Row colPercent={5}>
    <Col size={5} offset={1}>
      <Text>
        First Column
      </Text>
    </Col>
    <Col sm={6} md={4} mdOffset={2} lg={3} lgOffset={3} >
      <Text>
        Second Column
      </Text>
    </Col>
</Row>
```

**colPercent** may be supplied as prop to Row. Possible values are 0 to 100. This number defines the width of a single grid column as a percentage of the row element's width. If not supplied on the row, the width of child Columns will default to 100% of the computed or absolute width of the row. 

**size** may be supplied as prop to Column. Possible values is 0 to Infinity. This number defines how many grid columns wide the given column element should be. If `size` is not supplied it defaults to 1. If the column is without a parent Row or colPercent is not specified on parent Row the width of the column will default to 100% of the computed or absolute width of its parent. Since `size` accepts any number from 0 to Infinity, you can make your layout column as many grid columns wide as you want, extending beyond the screen width if **norwarp** prop is set on the row. If **nowrap** is not set on the row, the column will wrap. 

**sm**, **md**, and **lg** are device-size-dependent 'size' values that are applicable to columns.

**offset**, **smOffset**, **mdOffset** and **lgOffset** - Accepts any number. This number defines the marginLeft (or marginRight in csase of RTL mode) for the column in terms of the number of grid columns. Since grid columns have their parent row's justifyContent as flex-start (by design) and their alignItems set to flex-start (or flex-end for RTL mode) content in offsetted columns will snap to grid (in both LTR and RTL modes.) Offset values can also be negative, too. Column `offset` value defaults to 0. When using an offset rule in LTR or RTL modes, marginLeft and marginRight in style prop will be overridden, respectively.  

**vAlign** may be supplied as prop to Column to vertically align the elements and/or rows within it. Possible values are: middle, top, bottom, space and distribute. Default is top.

**vAlign** may also be supplied as prop to Row to align the columns within it in the vertical direction. Possible values are: top, middle, bottom or fill. Default is top.

**hAlign** may be supplied as prop to Row to align the columns within it in the horizontal direction. Possible values are: center, left, right, space and distribute. Default is left.

**hAlign** may also be supplied as prop to Column to align its rows and/or elements within it in the horizontal direction. Possible values are: center, left, right, and fill. Default is left.

**rtl** may be supplied as prop to Row to both reverse the order of columns (or elements) inside a row as well as to set alignX to 'right.' This is useful for Hebrew and Arabic layouts. 

**cell** may be supplied as prop to Row to set its height to 100% of the computed or absolute height of it's parent view. It will set `vAlign` on the row to 'fill' unless another `vAlign` value is explicitly supplied on the row. It will also set `hAlign` on all child columns to 'fill' unless another `hAlign` value is explicitly supplied on the column.

These make up the basic rules from which arbirarily complex layout behavior may emerge. 

### Sizing & Offsets

```
import {Column as Col, Row} from 'react-native-responsive-grid';

<Row colPercent={7}>
    <Col sm={6} md={4} lg={3} smOffset={1} mdOffset={3} lgOffset={5}>
        <Text>First Column</Text>
    </Col>
</Row>
```

There are currently four size props for `Column`. `size`, `sm`, `md`, and `lg`. The first one, `size`, applies to all screen sizes. The values indicate how many columns wide the column should be. The intrinsic column width is defined by percentage relative to the row as parent. 

If you're nesting a column inside a row which is inside another column that is inside another row as below:

```
<Row colPercent={5}>
    <Col size={10}>
      <Row colPercent={5}>
        <Col size={10}>
          <Text>
            This column is 25% of the width of the top level row
          </Text>
        </Col>
      </Row>
    </Col>
</Row>

```

The nested column's size will be the column size value (size, sm, md, lg) times the colPercent of its parent row, so, in effect, we get nested percentages, e.g. 50% of parent row's width which is 50% of its parent row's width means the nested column is 25% of the top level row's width. 

This nested percentages model affects offsets since they are a multiple of the grid column width. 

There are also four offset props for `Column`. `offset`, `smOffset`, `mdOffset`, and `lgOffset`. The first one, `offset`, applies to all screen sizes. Offset values can be negative, too, and that's often used when rightAlign is supplied as prop to the column, so that content of offsetted column will snap to grid in the right-to-left direction just as it would in the left-to-right direction with a positive offset value.

The screen-size-prefixed size and offset props refer to the screen sizes they are active on (taking device pixel ratio into consideration.) 

|Prop |Screen Size|Real World Device   |
|---|---|---|
|sm | < 768px   | Phone (iPhone)  |
|md | 768px -  1023px  | Normal Tablet (iPad)  |
|lg | >=1024px  |  Big Tablet (iPad Pro)|

Example: 

```
import {Column as Col, Row} from 'react-native-responsive-grid';

<Row colPercent={10}>
    <Col sm={5} md={3.3333} lg={2.5}>
        <Text>First Column</Text>
    </Col>
</Row>
```

On a phone the Column would take up 50% of the row's width.
On a normal tablet the Column would take up 33.333% of the row's width.
On a big tablet the Column would take up 25% of the row's width.

### Hidden props

```
import {Column as Col, Row} from 'react-native-responsive-grid';

<Row colPercent={15}>
    <Col smHidden>
        <Text>First Column</Text>
    </Col>
</Row>
```

In this example the column and all of it's children will be hidden on small screens like phones, but it will appear on bigger screens like tablets. These props can be applied to either the row as a whole or to individual columns within it. 

Every screen size has a hidden prop associated with it.

Hidden props are all booleans. They default to false.

## Real world example using relative size and offset props:

![demo](https://s2.postimg.org/im8oxf195/Screen_Shot_2017-04-17_at_2.59.31_PM.png)

### Navbar styles (for ex-navigation)

```
  in route's HOC:

  static route = {
      navigationBar: {
        title: 'Home',
        renderTitle: (route, props) => {
          return (
          <Row fill vAlign='center'>
            <Col hAlign='center'>
              <Image style={styles.titleImage} source={require('./assets/logo.png')}/>
            </Col>
          </Row>)
        },
        renderRight: (route, props) => {

          const { config: { eventEmitter }  } = route;

          return (<Row fill vAlign='center'>
            <Col hAlign='right'>
              <Button 
                title="LOG IN"
                color="#0A0A0A"
                onPress={() => {
                    eventEmitter.emit('openModal')
                  }
                }
              ></Button>
            </Col>
          </Row>)
        },
        backgroundColor: "#fff"
      }
    }

    ...
    
    // in styles:

    titleImage: {
      width: 120,
      height: 24,
      resizeMode: 'stretch' 
    }
```

Note that in the markup below the right arrow icons have padding on the right and left (they should not but I guess they were converted from vector to image and that's how they got their extra padding) so a good way to deal with that is not by using fractional offset value as that will change with screen size while the font remains the same size which would misalign the icons relative to the right-aligned text like SEE ALL, ADD MORE and the start icon (which has no padding in it) -- the right to compensate for icons that have padding in them is by using absolute pixels in the style prob, e.g. left: 6. That is unless your font is responsive, in which case using fractional offset would be the right way.

```
    <Row colPercent={6} style={{paddingTop: '11%', paddingBottom: '4%', backgroundColor: '#f3f3f3', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1} >
          <Text style={{fontWeight: 'bold', fontSize: 18, color: 'black'}}>
          PREVIOUS ORDERS
          </Text>
        </Col>
        <Col size={5} hAlign='right'>
          <Text style={{ fontSize: 16, color: '#BD1206'}}>
            SEE ALL
          </Text>
        </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '6%', paddingBottom: '6%', backgroundColor: 'white', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1} >
            <Text style={{fontSize: 15, color: '#BD1206', fontWeight:'bold'}}>February 28, 2017</Text>
            <Row colPercent={5}>
              <Col size={1}>
                <FontAwesome name='shopping-cart' size={17} color='gray'/>
              </Col>
              <Col size={12} offset={.5}>
                <Text style={{fontSize: 12, color: 'gray', lineHeight: 20}}>TAKEOUT ORDER</Text>
              </Col>
            </Row>
          <Text style={{fontSize: 16, color: '#0a0a0a'}}>Grilld Cheese Sandwich</Text>
          <Text style={{fontSize: 16, color: '#0a0a0a'}}>Key Lime Pie</Text>                                                                             
        </Col>
        <Col size={5} hAlign='right'>
          <MaterialIcons name="keyboard-arrow-right" size={28} color="#BD1206" style={{left: 5}} />
        </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '6%', paddingBottom: '6%', backgroundColor: 'white', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1}>
            <Text style={{fontSize: 15, color: '#BD1206', fontWeight:'bold'}}>March 8, 2017</Text>
            <Row colPercent={5}>
              <Col size={1}>
                <FontAwesome name='cutlery' size={17} color='gray'/>
              </Col>
              <Col size={12} offset={.5}>
                <Text style={{fontSize: 12, color: 'gray', lineHeight: 20}}>DINE-IN ORDER</Text>
              </Col>
            </Row>
          <Text style={{fontSize: 16, color: '#0a0a0a'}}>Linguini Alfredo</Text>                                                                          
        </Col>
        <Col size={5} hAlign='right'>
          <MaterialIcons name="keyboard-arrow-right" size={28} color="#BD1206" style={{left: 5}}/>
        </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '6%', paddingBottom: '6%', backgroundColor: 'white', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1}>
            <Text style={{fontSize: 15, color: '#BD1206', fontWeight:'bold'}}>March 9, 2017</Text>
            <Row colPercent={5}>
              <Col size={1}>
                <FontAwesome name='cutlery' size={17} color='gray'/>
              </Col>
              <Col size={12} offset={.5}>
                <Text style={{fontSize: 12, color: 'gray', lineHeight: 20}}>TAKEOUT ORDER</Text>
              </Col>
            </Row>
          <Text style={{fontSize: 16, color: '#0a0a0a'}}>Double Cheese Burger</Text>                                                                          
        </Col>
        <Col size={5} hAlign='right'>
          <MaterialIcons name="keyboard-arrow-right" size={28} color="#BD1206" style={{left: 5}}/>
        </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '11%', paddingBottom: '4%', backgroundColor: '#f3f3f3', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1}>
          <Text style={{fontWeight: 'bold', fontSize: 18, color: 'black'}}>
          FAVORITE ITEMS
          </Text>
        </Col>
        <Col size={5} hAlign='right'>
          <Text style={{ fontSize: 16, color: '#BD1206'}}>
          ADD MORE
          </Text>
        </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '6%', paddingBottom: '6%', backgroundColor: 'white', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
      <Col size={10} offset={1}>
        <Text style={{fontSize: 16, color: 'black'}}>
        Linguini Alfredo
        </Text>
      </Col>
      <Col size={5} hAlign='right'>
        <FontAwesome name='star' size={24} color='#BD1206'/>
      </Col>
    </Row>

    <Row colPercent={6} style={{paddingTop: '6%', paddingBottom: '6%', backgroundColor: 'white', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
      <Col size={10} offset={1}>
        <Text style={{fontSize: 16, color: 'black'}}>
        Double Cheese Burger
        </Text>
      </Col>
      <Col size={5} hAlign='right'>
        <FontAwesome name='star' size={24} color='#BD1206'/>
      </Col>
    </Row>
```

## RTL Support

This is intended for Hebrew and Arabic layouts, which are right to left.

Notice the reversed order of the Text relative to the physical order in the markup. Also notice that columns are justified as flex-end within the row and their content is rightAligned (except for the second column which is explicitly leftAligned to mimic the rightAligned behavior in normal ltr layout)

![demo](https://s18.postimg.org/gr89vaghl/Screen_Shot_2017-04-07_at_6.47.22_PM.png)

### Normal LTR Markup 

```
    <Row colPercent={6} style={{paddingTop: '11%', paddingBottom: '4%', backgroundColor: '#f3f3f3', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={1} >
          <Text style={{fontWeight: 'bold', fontSize: 18, color: 'black'}}>
          PREVIOUS ORDERS
          </Text>
        </Col>
        <Col size={5} hAlign='right'>
          <Text style={{ fontSize: 16, color: '#BD1206'}}>
            SEE ALL
          </Text>
        </Col>
    </Row>
```

### RTL Markup

Notice the offset values work in RTL direction now. The addition of .7 offset is to mimic the fact that the left margin in the LTR layout is smaller than the right margin in that layout, whereas it's the opposite in the RTL direction. So the .7 offset is used in RTL layout instead of the 1 offset, so alignment is identical. 

```
    <Row rtl colPercent={6} style={{paddingTop: '11%', paddingBottom: '4%', backgroundColor: '#f3f3f3', borderBottomColor: 'lightgray', borderBottomWidth: 1}}>
        <Col size={10} offset={.7} >
          <Text style={{fontWeight: 'bold', fontSize: 18, color: 'black'}}>
          PREVIOUS ORDERS
          </Text>
        </Col>
        <Col size={5} hAlign='left'>
          <Text style={{ fontSize: 16, color: '#BD1206'}}>
            SEE ALL
          </Text>
        </Col>
    </Row>
```

## Another Real-World Example (Advanced)

![demo](https://s8.postimg.org/7t9wefrrp/Screen_Shot_2017-04-17_at_2.59.00_PM.png)

```
    <Modal
      animationType="fade"
      transparent={true}
      visible={this.state.modalVisible}
      onRequestClose={() => this.close()}
      >
      <Row cell style={[{padding: 20}, modalBackgroundStyle]}>
        <Col hAlign='center' style={{backgroundColor: "#f3f3f3", padding: 20}}>
              
              <Row nowrap colPercent={10} style={{height: 80}}>
                <Col size={3.333} offset={3.333} hAlign='center' >
                  <Text>
                    <Image source={require('./assets/logo-login.png')} style={styles.logoImage}/>
                  </Text>
                </Col>
                <Col size={3.333} hAlign='right'>
                  <TouchableHighlight activeOpacity={0.5} underlayColor='#f3f3f3' onPress={() => this.close()}>
                      <FontAwesome
                        name="close"
                        size={28}
                        color="#d0d0d0"
                      />
                  </TouchableHighlight>
                </Col>
              </Row>

              <Row vAlign='middle' style={{height: 50}}>
                <Text style={{fontFamily: 'lubalin-graph-regular', fontSize: 16}}>LOG IN TO YOUR ACCOUNT</Text>
              </Row>

              <Row nowrap vAlign='middle' style={{height: 55}}>
                <Col style={{height: 40, borderStyle: 'solid', borderColor: '#a0a0a0', borderWidth: 1, borderRadius: 2, padding: 10}}>
                  <Row colPercent={3}>
                    <Col size={3.333} >
                        <FontAwesome name='envelope' size={20} color='#BD1206'/>
                    </Col>
                    <Col size={30}>
                        <TextInput placeholder='Email' style={{flex: 1}}/>
                    </Col>
                  </Row>
                </Col>
              </Row>

              <Row vAlign='middle' style={{height: 55}}>
                <Col style={{height: 40, borderStyle: 'solid', borderColor: '#a0a0a0', borderWidth: 1, borderRadius: 2, padding: 10}}>
                  <Row colPercent={3}>
                    <Col size={3.333} >
                        <FontAwesome name='envelope' size={20} color='#BD1206'/>
                    </Col>
                    <Col size={30}>
                        <TextInput secureTextEntry={true} placeholder='Password' style={{flex: 1}}/>
                    </Col>
                  </Row>
                </Col>
              </Row>

              <Row hAlign='center' style={{height: 12}}>
                <Text style={{color: '#BD1206', fontSize: 12}}>
                Forgot password?
                </Text>
              </Row>

              <Row style={{ height: 20}}></Row>

              <Row nowrap vAlign='middle' style={{height: 60}}>
                <Col hAlign='fill'>
                  <Row hAlign='center' vAlign='middle' style={{height: 36, borderRadius: 20, backgroundColor: '#BD1206'}}>
                    <TouchableHighlight activeOpacity={0.5} underlayColor='transparent' onPress={() => this.login()}>
                      <Text style={{color: 'white', fontSize: 16, fontWeight: 'bold'}}>LOG IN</Text>
                    </TouchableHighlight>
                  </Row>
                </Col>
              </Row>

              <Row vAlign='middle' style={{height: 60}}>
                <Col style={{height: 1,  backgroundColor: '#a0a0a0'}}>
                </Col>
              </Row>

              <Row style={{height: 60}}>
                <Col vAlign='middle' style={{height: 36, borderRadius: 20, backgroundColor: '#3B5998'}}>
                  <Row nowrap colPercent={3}>
                    <Col size={3.333} offset={2}>  
                      <FontAwesome name='facebook' size={20} color='#f3f3f3'/>
                    </Col> 
                    <Col size={27} offset={1}>
                      <Text style={{color: 'white', fontSize: 16, fontWeight: 'bold'}}> 
                        LOG IN WITH FACEBOOK
                      </Text>
                    </Col>
                  </Row>
                </Col>
              </Row>

        </Col>
      </Row>
    </Modal>
```
## Making Simple Easy

Only tricky part about using this super robust grid is the ability to do basic mental arithmetic:

```
    <Row nowrap colPercent={3}>
      <Col size={3.333} offset={2}>  
        <FontAwesome name='facebook' size={20} color='#f3f3f3'/>
      </Col> 
      <Col size={27} offset={1}>
        <Text style={{color: 'white', fontSize: 16, fontWeight: 'bold'}}> 
          LOG IN WITH FACEBOOK
        </Text>
      </Col>
    </Row>
```

This says that we're going to have a row where each grid column is 3% of the width of the screen (since this is a top level row, not nested) 

We must make sure all the columns which have their width as a multiple of that 3% add up to 100% or less or else they extend beyond the screen.

The first column is 10% in width (3.333 * 3) and the second one is 81% (27 * 3) ... That adds up to 91%. The other 9% comes from the 2 column positions the first column is offset by, which is 6% (2 * 3) and the 1 column position the second column in offset by which is 3% (1 * 3)

All the calculations are in percentages and multiples of a percentage. This way we can control our grid offsets and layout column sizes precisely and independently. 
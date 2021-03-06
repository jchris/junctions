# An Example

This example puts everything together that you'd learned so far.

- It defines three [Screens](ScreensAndLinks.md), with a [JunctionSet](Junctions.md) for each screen
- It uses [locate](ScreensAndLinks.md) and the react-junctions [Link](ScreensAndLinks.md) component to create links to those screens
- It chooses which screen to render based on the current [RouteSet](Routes.md)
- It creates a [history](Locations.md), re-rendering the root React component when it emits a new [Location](Location.md).

You can see this example [live](jamesknelson.com/react-junctions-example).

```jsx
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" Main="width=device-width">
  <title>Raw React/Junctions Demo</title>

  <script src="https://unpkg.com/babel-core@5.8.38/browser.min.js"></script>

  <script src="https://unpkg.com/react@15.3.2/dist/react.js"></script>
  <script src="https://unpkg.com/react-dom@15.3.2/dist/react-dom.js"></script>
  <script src="https://unpkg.com/history@4.3.0/umd/history.min.js"></script>
  <script src="https://unpkg.com/junctions@0.1.0/dist/junctions.js"></script>
  <script src="https://unpkg.com/react-junctions@0.1.0/dist/react-junctions.js"></script>
</head>
<body>
  <div id="app"></div>
  <script type="text/babel">
    const history = History.createBrowserHistory()
    const { JunctionSet, Junction, Branch, Param, Serializer, createRoute } = Junctions
    const { Link } = ReactJunctions


    /*
     * Dashboard Screen
     */
    
    const DashboardScreen = React.createClass({
      render: function() {
        return (
          <div>
            <h2>Dashboard</h2>
          </div>
        )
      },
    })


    /*
     * Contacts Screen
     */

    const ContactsMain = Junction({
      list: Branch({
        default: true,
      }),

      id: Branch({
        path: '/:id',
        params: {
          id: Param({ required: true }),
        }
      }),
    })

    const ContactsModal = Junction({
      add: Branch({}),
    })


    const ContactsScreen = React.createClass({
      statics: {
        junctionSet: JunctionSet({
          main: ContactsMain,
          modal: ContactsModal,
        })
      },

      render: function() {
        const locate = this.props.locate
        const { page, pageSize } = this.props.params
        const { main, modal } = this.props.routes

        const detail = 
          main &&
          main.branch == ContactsMain.id &&
          <div>
            <h3>Contact #{main.params.id}</h3>
          </div>

        const modalElement =
          modal &&
          <div style={{
            position: 'fixed',
            left: 0,
            top: 0,
            bottom: 0,
            right: 0,
            backgroundColor: 'rgba(0, 0, 0, 0.5)',
          }}>
            <div style={{backgroundColor: 'white'}}>
              <h1>Add</h1>
              <nav>
                <Link to={ locate(main) } history={history}>Close</Link>
              </nav>
            </div>
          </div>

        return (
          <div>
            <div>
              <h2>Contacts Page {this.props.params.page}</h2>
              <nav>
                <Link to={ locate(main, createRoute(ContactsModal.add)) } history={history}>Add</Link>
              </nav>
              <ul>
                <li><Link to={ locate(createRoute(ContactsMain.id, { id: 'james-nelson' })) } history={history}>James K Nelson</Link></li>
              </ul>
            </div>
            {detail}
            {modalElement}
          </div>
        )
      },
    })


    /*
     * App Screen
     */

    const AppMain = Junction({
      dashboard: Branch({
        default: true,
        data: {
          Component: DashboardScreen,
        },
      }),
      contacts: Branch({
        path: '/contacts',
        data: {
          Component: ContactsScreen,
        },
        children: ContactsScreen.junctionSet,
        params: {
          page: Param({
            default: 1,
            serializer: Serializer({ serialize: x => String(x), deserialize: x => x === '' ? null : parseInt(x) })
          }),
          pageSize: Param({
            default: 20,
            serializer: Serializer({ serialize: x => String(x), deserialize: x => x === '' ? null : parseInt(x) })
          }),
        },
      }),
    })


    const AppScreen = React.createClass({
      statics: {
        junctionSet: JunctionSet({
          main: AppMain,
        })
      },

      render: function() {
        const locate = this.props.locate
        const { main } = this.props.routes

        return (
          <div>
            <p>
              Hi! This is a demo for the <a href="https://github.com/jamesknelson/junctions">Junctions</a> routing system for React.
            </p>
            <p>
              The source is all contained in old-school script tags. View source to get smarter. Also check out the <a href="">Junction Diagram</a>.
            </p>
            <hr />
            <nav>
              <Link to={ locate(createRoute(AppMain.dashboard)) } history={history}>Dashboard</Link>
              <Link to={ locate(createRoute(AppMain.contacts)) } history={history}>Contacts</Link>
            </nav>
            <main.data.Component
                routes={main.children}
                locate={main.locate}
                params={main.params}
            />
          </div>
        )
      },
    })


    /*
     * Entry Point
     */

    const baseLocation = { pathname: '/' }
    const converter = Junctions.createConverter(AppScreen.junctionSet, baseLocation)
    
    function render(routes) {
      ReactDOM.render(
        <AppScreen
          routes={routes}
          locate={converter.getLocation}
        />,
        document.getElementById('app')
      )
    }

    function handleLocationChange(location) {
      const routes = converter.getRouteSet(location)
      const canonicalLocation = converter.getLocation(routes)

      if (!Junctions.locationsEqual(location, canonicalLocation)) {
        history.replace(canonicalLocation)
      }

      render(routes)
    }

    handleLocationChange(history.location)
    history.listen(handleLocationChange)
  </script>
</body>
</html>
```

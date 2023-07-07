
# Get rid of useEffect for initial data fetch, take advantage of react-router 6 hook: useLoaderData

I will be trying to get rid of  `useEffect`  on initial render with the help of  `react-router`  . React router v6 came out with some cool new hooks that will help to ease the process of loading data, submitting forms. Thanks to  [Remix](https://remix.run/)  team.

As mentioned by React team: UseEffect is for  `synchronisation`  not for  `lifecycle methods`.

In our react app we usually call the api in  `useEffect`  as example given below. But in React 18 this useEffect is called twice and some developers think this is some bug in react as this occurs only in development mode, believe me this is not bug.

    import React, { useState, useEffect } from 'react';  
      
    const Homepage = () => {  
      const [data, setData] = useState(null);  
      
      useEffect(() => {  
        // Fetch the data from the API  
        fetch('https://example.com/api/data')  
          .then((response) => response.json())  
          .then((json) => setData(json));  
      }, []);  
      
      // Return a loading message while the data is being fetched  
      if (data === null) {  
        return <p>Loading...</p>;  
      }  
      
      // Render the data when it is available  
      return <pre>{JSON.stringify(data, null, 2)}</pre>;  
    };  
      
    export default Homepage;

Now to get rid of this  `useEffect`  , we will use  `useLoaderData`  hook provided by  `react-router`  . This hook is available only in v6+. To harness the power of this hook we need to change the way we define our routes.

    import {  createBrowserRouter, defer } from "react-router-dom";  
      
    const router = createBrowserRouter([  
      {  
        path: "/",  
        element: <Homepage/>,  
        loader: async ({ request, params }) => {  
           const data =  fetch('https://example.com/api/data')  
            return defer({  
               results: data,  
             })  
        },  
      },  
    ]);

Whenever  `/homepage`  route will be invoked the  `loader`  function will be called and the data returned from this request can be used by the  `Homepage`  component to get the returned data. So now component will look something like this:

    import React from 'react';  
    import { useLoaderData } from "react-router-dom";  
      
    const Homepage = () => {  
      const data = useLoaderData()  
      
      // Render the data when it is available  
      return <pre>{JSON.stringify(data, null, 2)}</pre>;  
    };  
      
    export default Homepage;

With the help of this hook now we are able to get rid of  `useEffect`  and  `useState`  . But now you would be wondering  `what if we want to show loader while data is being fetched`  . We can use  `Await`  component provided by  `react-router`  to show loading. So now our component will look like this:

    import React from 'react';  
    import { useLoaderData, Await } from "react-router-dom";  
      
    const Homepage = () => {  
      const data = useLoaderData()  
      
      // Render the data when it is available  
      return <React.Suspense  
      fallback={<p>Loading data...</p>}  
    >  
      <Await  
        resolve={data.result}  
        errorElement={  
          <p>Error loading data</p>  
        }  
      >  
        {(result) => (  
          <pre>{JSON.stringify(results, null, 2)}</pre>  
        )}  
      </Await>  
    </React.Suspense>  
    };  
      
    export default Homepage;

Thanks for visiting!

Questions:
* how to call multiple apis?
* how to use api call status?

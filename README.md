# Instachat-
a messaging app just like Instagram
{

  "name": "messenger-starter",

  "version": "1.0.0",

  "main": "node_modules/expo/AppEntry.js",

  "scripts": { "start": "expo start" },

  "dependencies": {

    "expo": "~48.0.0",

    "expo-av": "~13.0.0",

    "expo-image-picker": "~14.0.1",

    "firebase": "^10.0.0",

    "react": "18.2.0",

    "react-native": "0.72.0",

    "@react-navigation/native": "^6.1.6",

    "@react-navigation/native-stack": "^6.9.12",

    "@react-navigation/bottom-tabs": "^6.5.7",

    "react-native-gesture-handler": "~2.9.0",

    "react-native-safe-area-context": "4.5.0",

    "react-native-screens": "~3.20.0",

    "react-native-gifted-chat": "^1.0.0",

    "uuid": "^9.0.0"

  }

}

// firebase.js

import { initializeApp } from "firebase/app";

import { getAuth } from "firebase/auth";

import { getFirestore, serverTimestamp } from "firebase/firestore";

import { getStorage } from "firebase/storage";

import { getMessaging } from "firebase/messaging";


const firebaseConfig = {

  apiKey: "YOUR_API_KEY",

  authDomain: "your-app.firebaseapp.com",

  projectId: "your-app",

  storageBucket: "your-app.appspot.com",

  messagingSenderId: "SENDER_ID",

  appId: "APP_ID"

};


const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);

export const db = getFirestore(app);

export const storage = getStorage(app);

export const ts = serverTimestamp;

export const messaging = getMessaging ? getMessaging(app) : null;

// App.js

import React, { useEffect, useState } from "react";

import { NavigationContainer } from "@react-navigation/native";

import { createNativeStackNavigator } from "@react-navigation/native-stack";

import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";

import AuthScreen from "./screens/AuthScreen";

import FeedScreen from "./screens/FeedScreen";

import PostScreen from "./screens/PostScreen";

import ProfileScreen from "./screens/ProfileScreen";

import ChatListScreen from "./screens/ChatListScreen";

import ChatScreen from "./screens/ChatScreen";

import StoriesScreen from "./screens/StoriesScreen";

import ReelsScreen from "./screens/ReelsScreen";

import { onAuthStateChanged } from "firebase/auth";

import { auth } from "./firebase";


const Stack = createNativeStackNavigator();

const Tab = createBottomTabNavigator();


function MainTabs() {

  return (

    <Tab.Navigator>

      <Tab.Screen name="Feed" component={FeedScreen} />

      <Tab.Screen name="Post" component={PostScreen} />

      <Tab.Screen name="Stories" component={StoriesScreen} />

      <Tab.Screen name="Reels" component={ReelsScreen} />

      <Tab.Screen name="Profile" component={ProfileScreen} />

      <Tab.Screen name="Chats" component={ChatListScreen} />

    </Tab.Navigator>

  );

}


export default function App() {

  const [user, setUser] = useState(null);

  useEffect(() => {

    const unsub = onAuthStateChanged(auth, (u) => setUser(u));

    return () => unsub();

  }, []);


  return (

    <NavigationContainer>

      <Stack.Navigator screenOptions={{ headerShown: false }}>

        {!user ? (

          <Stack.Screen name="Auth" component={AuthScreen} />

        ) : (

          <>

            <Stack.Screen name="Main" component={MainTabs} />

            <Stack.Screen name="Chat" component={ChatScreen} />

          </>

        )}

      </Stack.Navigator>

    </NavigationContainer>

  );

}

// screens/AuthScreen.js

import React, { useState } from "react";

import { View, TextInput, Button, Text } from "react-native";

import { createUserWithEmailAndPassword, signInWithEmailAndPassword, updateProfile } from "firebase/auth";

import { auth, db } from "../firebase";

import { doc, setDoc } from "firebase/firestore";


export default function AuthScreen() {

  const [email, setEmail] = useState("");

  const [pass, setPass] = useState("");

  const [name, setName] = useState("");


  const register = async () => {

    const res = await createUserWithEmailAndPassword(auth, email, pass);

    await updateProfile(res.user, { displayName: name });

    // create user doc

    await setDoc(doc(db, "users", res.user.uid), {

      uid: res.user.uid,

      name,

      email,

      bio: "",

      createdAt: new Date()

    });

  };


  const login = async () => {

    await signInWithEmailAndPassword(auth, email, pass);

  };


  return (

    <View style={{ padding: 20, marginTop: 80 }}>

      <TextInput placeholder="Name" value={name} onChangeText={setName} style={{borderBottomWidth:1, marginBottom:10}} />

      <TextInput placeholder="Email" value={email} onChangeText={setEmail} style={{borderBottomWidth:1, marginBottom:10}} />

      <TextInput placeholder="Password" secureTextEntry value={pass} onChangeText={setPass} style={{borderBottomWidth:1, marginBottom:10}} />

      <Button title="Register" onPress={register} />

      <View style={{ height: 10 }} />

      <Button title="Login" onPress={login} />

      <Text style={{marginTop:20, color:'#666'}}>Use this screen to create account or login</Text>

    </View>

  );

}

// screens/ProfileScreen.js

import React, { useEffect, useState } from "react";

import { View, Text, Button, Image } from "react-native";

import { auth, db, storage } from "../firebase";

import { doc, getDoc, updateDoc, arrayUnion, arrayRemove } from "firebase/firestore";


export default function ProfileScreen({ navigation, route }) {

  const [profile, setProfile] = useState(null);

  const uid = route.params?.uid || auth.currentUser.uid;


  useEffect(() => {

    (async () => {

      const snap = await getDoc(doc(db, "users", uid));

      setProfile(snap.data());

    })();

  }, [uid]);


  if (!profile) return <View><Text>Loading...</Text></View>;


  const isMe = uid === auth.currentUser.uid;

  const isFollowing = (profile.followers || []).includes(auth.currentUser.uid);


  const toggleFollow = async () => {

    const userRef = doc(db, "users", uid);

    if (isFollowing) {

      await updateDoc(userRef, { followers: arrayRemove(auth.currentUser.uid) });

      await updateDoc(doc(db, "users", auth.currentUser.uid), { following: arrayRemove(uid) });

    } else {

      await updateDoc(userRef, { followers: arrayUnion(auth.currentUser.uid) });

      await updateDoc(doc(db, "users", auth.currentUser.uid), { following: arrayUnion(uid) });

    }

  };


  return (

    <View style={{ padding: 20 }}>

      <Image source={{uri: profile.photoUrl || 'https://via.placeholder.com/100'}} style={{width:100,height:100,borderRadius:50}} />

      <Text style={{fontSize:20}}>{profile.name}</Text>

      <Text>{profile.bio}</Text>

      {!isMe && <Button title={isFollowing ? "Unfollow" : "Follow"} onPress={toggleFollow} />}

    </View>

  );

}

// screens/FeedScreen.js

import React, { useEffect, useState } from "react";

import { View, FlatList } from "react-native";

import { collection, onSnapshot, query, orderBy } from "firebase/firestore";

import { db } from "../firebase";

import PostCard from "../components/PostCard";


export default function FeedScreen() {

  const [posts, setPosts] = useState([]);


  useEffect(() => {

    const q = query(collection(db, "posts"), orderBy("createdAt", "desc"));

    const unsub = onSnapshot(q, (snap) => {

      setPosts(snap.docs.map(d => ({ id: d.id, ...d.data() })));

    });

    return () => unsub();

  }, []);


  return (

    <View style={{flex:1}}>

      <FlatList data={posts} keyExtractor={i=>i.id} renderItem={({item})=> <PostCard post={item} />} />

    </View>

  );

}

// components/PostCard.js

import React, { useState } from "react";

import { View, Text, Image, Button } from "react-native";

import { doc, updateDoc, arrayUnion, arrayRemove } from "firebase/firestore";

import { db, auth } from "../firebase";


export default function PostCard({ post }) {

  const [liked, setLiked] = useState(post.likes?.includes(auth.currentUser?.uid));

  const toggleLike = async () => {

    const ref = doc(db, "posts", post.id);

    if (liked) {

      await updateDoc(ref, { likes: arrayRemove(auth.currentUser.uid) });

    } else {

      await updateDoc(ref, { likes: arrayUnion(auth.currentUser.uid) });

    }

    setLiked(!liked);

  };


  return (

    <View style={{margin:10, borderWidth:1, borderColor:'#ddd'}}>

      <Image source={{uri: post.imageUrl}} style={{width:'100%', height:300}} />

      <View style={{padding:8}}>

        <Button title={liked ? "❤️ Unlike" : "🤍 Like"} onPress={toggleLike} />

        <Text>{post.likes?.length || 0} likes</Text>

        <Text>{post.caption}</Text>

      </View>

    </View>

  );

}

// screens/PostScreen.js

import React, { useState } from "react";

import { View, Button, Image, TextInput } from "react-native";

import * as ImagePicker from "expo-image-picker";

import { ref, uploadBytes, getDownloadURL } from "firebase/storage";

import { storage, db, auth, ts } from "../firebase";

import { collection, addDoc } from "firebase/firestore";

import { v4 as uuidv4 } from "uuid";


export default function PostScreen() {

  const [image, setImage] = useState(null);

  const [caption, setCaption] = useState("");


  const pick = async () => {

    const p = await ImagePicker.launchImageLibraryAsync({ quality:0.7 });

    if (p.canceled) return;

    setImage(p.assets[0].uri);

  };


  const upload = async () => {

    const blob = await (await fetch(image)).blob();

    const fileRef = ref(storage, `posts/${uuidv4()}.jpg`);

    await uploadBytes(fileRef, blob);

    const url = await getDownloadURL(fileRef);

    await addDoc(collection(db, "posts"), {

      imageUrl: url,

      caption,

      userId: auth.currentUser.uid,

      createdAt: ts(),

      likes: []

    });

    setImage(null); setCaption("");

  };


  return (

    <View style={{padding:20}}>

      <Button title="Pick Image" onPress={pick} />

      {image && <Image source={{uri:image}} style={{width:200,height:200,marginTop:10}} />}

      <TextInput placeholder="Caption" value={caption} onChangeText={setCaption} style={{borderBottomWidth:1, marginTop:10}} />

      <Button title="Upload Post" onPress={upload} disabled={!image} />

    </View>

  );

}

// screens/StoriesScreen.js

import React, { useEffect, useState } from "react";

import { View, FlatList, Image, TouchableOpacity } from "react-native";

import { collection, onSnapshot, query, where } from "firebase/firestore";

import { db, auth, ts } from "../firebase";

import { addDoc } from "firebase/firestore";

import * as ImagePicker from "expo-image-picker";

import { ref, uploadBytes, getDownloadURL } from "firebase/storage";

import { v4 as uuidv4 } from "uuid";

import { storage } from "../firebase";


export default function StoriesScreen() {

  const [stories, setStories] = useState([]);


  useEffect(() => {

    const q = query(collection(db, "stories"));

    const unsub = onSnapshot(q, snap => setStories(snap.docs.map(d=>({id:d.id,...d.data()}))));

    return () => unsub();

  }, []);


  const addStory = async () => {

    const p = await ImagePicker.launchImageLibraryAsync();

    if (p.canceled) return;

    const blob = await (await fetch(p.assets[0].uri)).blob();

    const fileRef = ref(storage, `stories/${uuidv4()}.jpg`);

    await uploadBytes(fileRef, blob);

    const url = await getDownloadURL(fileRef);

    await addDoc(collection(db, "stories"), {

      userId: auth.currentUser.uid,

      imageUrl: url,

      createdAt: ts(),

      expireAt: new Date(Date.now() + 24*60*60*1000)

    });

  };


  return (

    <View style={{padding:10}}>

      <Button title="Add Story" onPress={addStory} />

      <FlatList horizontal data={stories} keyExtractor={i=>i.id} renderItem={({item})=> (

        <TouchableOpacity>

          <Image source={{uri:item.imageUrl}} style={{width:80,height:140, margin:6}} />

        </TouchableOpacity>

      )} />

    </View>

  );

}

// screens/ChatListScreen.js

import React, { useEffect, useState } from "react";

import { View, FlatList, TouchableOpacity, Text } from "react-native";

import { collection, query, where, onSnapshot } from "firebase/firestore";

import { db, auth } from "../firebase";


export default function ChatListScreen({ navigation }) {

  const [chats, setChats] = useState([]);


  useEffect(() => {

    const q = query(collection(db, "chats"), where("participants", "array-contains", auth.currentUser.uid));

    const unsub = onSnapshot(q, snap => setChats(snap.docs.map(d=> ({id:d.id, ...d.data()}))));

    return () => unsub();

  }, []);


  return (

    <View style={{padding:10}}>

      <FlatList data={chats} keyExtractor={i=>i.id} renderItem={({item})=>(

        <TouchableOpacity onPress={()=>navigation.navigate('Chat',{chatId:item.id, other:item.participants.filter(p=>p!==auth.currentUser.uid)[0]})}>

          <Text style={{padding:10, borderBottomWidth:1}}>{item.title || item.participants.join(', ')}</Text>

        </TouchableOpacity>

      )} />

    </View>

  );

}

// screens/ChatScreen.js

import React, { useCallback, useEffect, useState } from "react";

import { GiftedChat } from "react-native-gifted-chat";

import { collection, addDoc, query, orderBy, onSnapshot } from "firebase/firestore";

import { db, ts, auth } from "../firebase";


export default function ChatScreen({ route }) {

  const { chatId } = route.params;

  const [messages, setMessages] = useState([]);


  useEffect(() => {

    const q = query(collection(db, "chats", chatId, "messages"), orderBy("createdAt", "desc"));

    const unsub = onSnapshot(q, snap => {

      setMessages(snap.docs.map(d => ({ _id: d.id, ...d.data(), createdAt: d.data().createdAt?.toDate?.() })));

    });

    return () => unsub();

  }, [chatId]);


  const onSend = useCallback(async (msgs=[]) => {

    const m = msgs[0];

    await addDoc(collection(db, "chats", chatId, "messages"), {

      text: m.text,

      user: { _id: auth.currentUser.uid, name: auth.currentUser.displayName },

      createdAt: ts()

    });

  }, [chatId]);


  return <GiftedChat messages={messages} onSend={onSend} user={{_id: auth.currentUser.uid}} />;

}

// create chat between A and B

await addDoc(collection(db,"chats"), {

  participants: [uidA, uidB],

  createdAt: ts()

});

// screens/ReelsScreen.js

import React, { useEffect, useState } from "react";

import { View, Button, FlatList } from "react-native";

import * as ImagePicker from "expo-image-picker";

import { ref, uploadBytes, getDownloadURL } from "firebase/storage";

import { storage, db, ts } from "../firebase";

import { collection, addDoc, onSnapshot, orderBy, query } from "firebase/firestore";

import { v4 as uuidv4 } from "uuid";

import { Video } from "expo-av";


export default function ReelsScreen() {

  const [reels, setReels] = useState([]);


  useEffect(() => {

    const q = query(collection(db, "reels"), orderBy("createdAt", "desc"));

    const unsub = onSnapshot(q, snap => setReels(snap.docs.map(d=>({id:d.id,...d.data()}))));

    return () => unsub();

  }, []);


  const pickVideo = async () => {

    const p = await ImagePicker.launchImageLibraryAsync({ mediaTypes: ImagePicker.MediaTypeOptions.Videos });

    if (p.canceled) return;

    const blob = await (await fetch(p.assets[0].uri)).blob();

    const fileRef = ref(storage, `reels/${uuidv4()}.mp4`);

    await uploadBytes(fileRef, blob);

    const url = await getDownloadURL(fileRef);

    await addDoc(collection(db, "reels"), { videoUrl: url, createdAt: ts(), userId: "me" });

  };


  return (

    <View style={{flex:1}}>

      <Button title="Upload Reel" onPress={pickVideo} />

      <FlatList data={reels} keyExtractor={i=>i.id} renderItem={({item})=>(

        <Video source={{uri:item.videoUrl}} style={{height:300}} shouldPlay isLooping resizeMode="cover" />

      )} />

    </View>

  );

}

// firestore.rules (dev only)

rules_version = '2';

service cloud.firestore {

  match /databases/{database}/documents {

    match /{document=**} {

      allow read, write: if request.auth != null;

    }

  }

}

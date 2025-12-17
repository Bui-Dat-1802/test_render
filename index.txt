require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");

const app = express();
app.use(express.json({ limit: "10mb" }));
const mongoURI =
    "mongodb+srv://20225274:Ss35uX7aXvMOVtJE@testweb.mgkmb2q.mongodb.net/2025274?retryWrites=true&w=majority";
mongoose
    .connect(mongoURI, { serverSelectionTimeoutMS: 5000 })
    .then(() => console.log("Connected to MongoDB Atlas"))
    .catch((err) => console.error("Connection error:", err));

mongoose.connection.on("connected", () => console.log("Mongoose connected to DB"));
mongoose.connection.on("error", (err) => console.error("Mongoose connection error:", err));
const userSchema = new mongoose.Schema(
    {
        name: { type: String, required: true },
        age: { type: Number, min: 0 },
        email: { type: String, required: true, unique: true },
        gender: { type: String, enum: ["Nam", "Nữ", "Khác"], required: true },
    },
    { timestamps: true }
);

const User = mongoose.model("User", userSchema, "Dat.BX225274");
app.use((req, res, next) => {
    if (mongoose.connection.readyState !== 1) {
        return res.status(503).json({ error: "MongoDB not connected" });
    }
    next();
});
app.post("/users", async (req, res) => {
    try {
        const user = await User.create(req.body);
        res.status(201).json(user);
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});
app.get("/users", async (req, res) => {
    try {
        const users = await User.find().lean();
        res.json(users);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});
app.put("/users/:id", async (req, res) => {
    try {
        const updated = await User.findByIdAndUpdate(req.params.id, req.body, { new: true, runValidators: true });
        if (!updated) return res.status(404).json({ message: "User not found" });
        res.json(updated);
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});
app.delete("/users/:id", async (req, res) => {
    try {
        const deleted = await User.findByIdAndDelete(req.params.id);
        if (!deleted) return res.status(404).json({ message: "User not found" });
        res.json({ message: "User deleted" });
    } catch (err) {
        res.status(400).json({ error: err.message });
    }
});
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

# Front-end-assessment-quicksell

import React, { useEffect, useState } from "react";
import axios from "axios";
import "./App.css";

const API_URL = "https://api.quicksell.co/v1/internal/frontend-assignment";

const priorityLevels = ["No priority", "Low", "Medium", "High", "Urgent"];

const App = () => {
  const [tickets, setTickets] = useState([]);
  const [users, setUsers] = useState([]);
  const [grouping, setGrouping] = useState("status");
  const [sorting, setSorting] = useState("priority");
  const [loading, setLoading] = useState(true);


  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await axios.get(API_URL);
        setTickets(response.data.tickets);
        setUsers(response.data.users);
        setLoading(false);
      } catch (error) {
        console.error("Error fetching data:", error);
        setLoading(false);
      }
    };
    fetchData();
  }, []);

 
  const groupTickets = () => {
    return tickets.reduce((groups, ticket) => {
      let key = "";
      if (grouping === "user") {
        const user = users.find((u) => u.id === ticket.userId);
        key = user ? user.name : "Unknown User";
      } else {
        key = ticket[grouping];
      }
      if (!groups[key]) {
        groups[key] = [];
      }
      groups[key].push(ticket);
      return groups;
    }, {});
  };

  const groupedTickets = groupTickets();

 
  const sortTickets = (tickets) => {
    return tickets.sort((a, b) => {
      if (sorting === "priority") {
        return b.priority - a.priority;
      } else if (sorting === "title") {
        return a.title.localeCompare(b.title);
      }
      return 0;
    });
  };

  return (
    <div className="app-container">
      <h1>Interactive Kanban Board</h1>

      <div className="controls">
        <label htmlFor="grouping">Group by:</label>
        <select
          id="grouping"
          value={grouping}
          onChange={(e) => setGrouping(e.target.value)}
        >
          <option value="status">Status</option>
          <option value="user">User</option>
          <option value="priority">Priority</option>
        </select>

        <label htmlFor="sorting">Sort by:</label>
        <select
          id="sorting"
          value={sorting}
          onChange={(e) => setSorting(e.target.value)}
        >
          <option value="priority">Priority</option>
          <option value="title">Title</option>
        </select>
      </div>

      {loading ? (
        <p>Loading tickets...</p>
      ) : (
        <div className="kanban-board">
          {Object.keys(groupedTickets).map((group) => (
            <div key={group} className="kanban-column">
              <h2>{group}</h2>
              {sortTickets(groupedTickets[group]).map((ticket) => (
                <div key={ticket.id} className="kanban-card">
                  <h3>{ticket.title}</h3>
                  <p><strong>Status:</strong> {ticket.status}</p>
                  <p><strong>User:</strong> {users.find((u) => u.id === ticket.userId)?.name || "Unknown"}</p>
                  <p><strong>Priority:</strong> {priorityLevels[ticket.priority]}</p>
                </div>
              ))}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default App;
